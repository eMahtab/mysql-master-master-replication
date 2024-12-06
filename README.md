# MySQL Bidirectional database replication between two master (master-master-replication)

This demo is a variation of https://github.com/eMahtab/mysql-master-slave-replication , where we had setup Master Slave replication from a single mysql master to a single mysql slave/replica.

In this demo we setup two MySQL instances (e.g. mysql-master-1 and mysql-master-2), both working as master. 
**mysql-master-1 replicates its data to mysql-master-2 and mysql-master-2 replicates its data to mysql-master-1.**

### !!! Prerequisite : You would need Docker installed on your system to follow along this demo.

We would run both mysql-master-1 and mysql-master-2 as docker containers, using docker compose.

## Step 1 : Create the Docker compose file and execute docker compose up

Below docker-compose-master-master.yml declares two services, named as mysql_master_1 and mysql_master_2 (in docker compose file actual containers are named as mysql-master-1 and mysql-master-2). We are using **mysql:8.0** as the docker image, and declare root user password as `toor` and create a test database.

**Make sure docker engine is running on your host machine before running the `docker compose -f docker-compose-master-master.yml up` command.**

```yml
---
version: "2"
services:
  mysql_master_1:
    image: mysql:8.0
    container_name: mysql-master-1
    volumes:
      - mysql-master-1-volume:/tmp
    command:
      [
        "mysqld",
        "--datadir=/tmp/master1/data",
        "--log-bin=bin.log",
        "--server-id=1"
      ]
    environment:
      &mysql-default-environment
      MYSQL_ROOT_PASSWORD: toor
      MYSQL_DATABASE: test
      MYSQL_USER: test_user
      MYSQL_PASSWORD: insecure
    ports:
      - "3308:3306"

  mysql_master_2:
    image: mysql:8.0
    container_name: mysql-master-2
    volumes:
      - mysql-master-2-volume:/tmp
    command:
      [
        "mysqld",
        "--datadir=/tmp/master2/data",
        "--log-bin=bin.log",
        "--server-id=2"
      ]
    environment: *mysql-default-environment
    ports:
      - "3309:3306"

volumes:
  mysql-master-1-volume:
  mysql-master-2-volume:
```

!["Starting two MySQL Instances as Docker Containers"](docker-compose-up.png?raw=true)

!["Two MySQL Instances as Docker Containers"](docker-containers.png?raw=true)

## Step 2 : Create Replication user on both the MySQL instances with REPLICATION SLAVE privilege
Next we need to create a replication user on both the MySQL instances `(mysql-master-1 and mysql-master-2)` and grant that user `REPLICATION SLAVE` privilege.
To do this, we first open the bash shell on each MySQL instance **`docker exec -it mysql-master-1 bash`** and connect to mysql **`mysql -uroot -ptoor`** running on the instance, then execute below mysql commands.
```sql
CREATE USER 'replicator'@'%' IDENTIFIED BY 'rotacilper';
GRANT REPLICATION SLAVE ON *.* TO 'replicator'@'%';
FLUSH PRIVILEGES;
```
Here we create a replication user called `replicator` with password `rotacilper` and grant this user **`REPLICATION SLAVE`** privilege, and finally flush privileges.

## Step 3 : Execute SHOW MASTER STATUS on both the MySQL instances
Get the master status, execute the command **`SHOW MASTER STATUS;`** on both the MySQL instances to find the Binlog file and position.

!["Get Master status"](create-replication-user-and-show-status.png?raw=true)

!["Get Master status"](create-replication-user-and-show-status-2.png?raw=true)

## Step 4 : Execute `CHANGE MASTER TO` and `START SLAVE;` command on both the MySQL instances
Next we need to execute **`CHANGE MASTER TO`** and `START SLAVE;` command on both the MySQL instances. Connect to each MySQL instance and execute below command, **_don't forget to update MASTER_LOG_FILE and MASTER_LOG_POS values_** which you got from executing SHOW MASTER STATUS command.

### On mysql-master-1
Update **_MASTER_LOG_FILE and MASTER_LOG_POS_** and then execute this on MySQL instance running on mysql-master-1 docker container

```sql
CHANGE MASTER TO
  MASTER_HOST='mysql_master_2',
  MASTER_PORT=3306,
  MASTER_USER='replicator',
  MASTER_PASSWORD='rotacilper',
  MASTER_LOG_FILE='[log_file_name_from_mysql-master-2]',
  MASTER_LOG_POS=[log_position_from_mysql-master-2],
  GET_MASTER_PUBLIC_KEY=1;
```

and then

```sql
START SLAVE;
```

### On mysql-master-2
Update **_MASTER_LOG_FILE and MASTER_LOG_POS_** and then execute this on MySQL instance running on mysql-master-2 docker container

```sql
CHANGE MASTER TO
  MASTER_HOST='mysql_master_1',
  MASTER_PORT=3306,
  MASTER_USER='replicator',
  MASTER_PASSWORD='rotacilper',
  MASTER_LOG_FILE='[log_file_name_from_mysql-master-1]',
  MASTER_LOG_POS=[log_position_from_mysql-master-1],
  GET_MASTER_PUBLIC_KEY=1;
```

and then

```sql
START SLAVE;
```

!["CHANGE MASTER TO"](change-master-to-and-start.png?raw=true)

!["CHANGE MASTER TO"](change-master-to-and-start-2.png?raw=true)

## Step 5 : Insert Records on mysql-master-1 and see Replication in action
By performing the steps 1 to 4, we have set the replication from mysql-master-1 to mysql-master-2 and vice versa, now whatever changes are done on any one of the MySQL instance will be replicated to other MySQL instance automatically. 

We create a `users` table under test database on `mysql-master-1` and insert 20 records in `users` table

```sql
USE test
CREATE TABLE users (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(255) NOT NULL);
INSERT INTO users (name)
VALUES
    ('Alice'),
    ('Bob'),
    ('Charlie'),
    ('David'),
    ('Eve'),
    ('Frank'),
    ('Grace'),
    ('Hannah'),
    ('Ivy'),
    ('Jack'),
    ('Karen'),
    ('Liam'),
    ('Mia'),
    ('Noah'),
    ('Olivia'),
    ('Paul'),
    ('Quincy'),
    ('Rachel'),
    ('Sophia'),
    ('Thomas');
```
!["Add Records on mysql instance"](add-records.png?raw=true)

Records are replicated on **_mysql-master-2_**

!["Records are replicated to other mysql instance"](check-records-on-other-mysql-instance.png?raw=true)

## Step 5a : Create and execute a MySQL procedure on mysql-master-1 and see Replication in action

```sql
DELIMITER $$
```
Next we create a procedure `insert_users` which inserts 10,000 records in `users` table
```sql
CREATE PROCEDURE insert_users()
BEGIN
    DECLARE i INT DEFAULT 1;
    WHILE i <= 10000 DO
        INSERT INTO users (name) VALUES (CONCAT('User_', i));
        SET i = i + 1;
    END WHILE;
END$$
```
```sql
DELIMITER ;
```

### And then execute the procedure

```sql
CALL insert_users();
```

!["Create and execute procedure on mysql-master-1"](create-and-execute-procedure.png?raw=true)

!["10,000 Records replicated on other MySQL instance"](records-replicated-on-other-master.png?raw=true)
