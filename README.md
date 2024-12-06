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
