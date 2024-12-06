# MySQL Bidirectional database replication between two master (master-master-replication)

This demo is a variation of https://github.com/eMahtab/mysql-master-slave-replication , where we had setup Master Slave replication from a single mysql master to a single mysql slave/replica.

In this demo we setup two MySQL instances (e.g. mysql-master-1 and mysql-master-2), both working as master. 
mysql-master-1 replicates its data to mysql-master-2 and mysql-master-2 replicates its data to mysql-master-1.

### !!! Prerequisite : You would need Docker installed on your system to follow along this demo.

We would run both mysql-master-1 and mysql-master-2 as docker containers, using docker compose.

## Step 1 : Create the Docker compose file and execute docker compose up

Below docker-compose-master-master.yml declares two services, named as mysql_master_1 and mysql_master_2 (in docker compose file actual containers are named as mysql-master-1 and mysql-master-2). We are using **mysql:8.0** as the docker image, and declare root user password as `toor` and create a test database.

**Make sure docker engine is running on your host machine before running the `docker compose up` command.**

