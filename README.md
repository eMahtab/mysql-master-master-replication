# MySQL Bidirectional database replication between two master (master-master-replication)

This demo is a variation of https://github.com/eMahtab/mysql-master-slave-replication , where we had setup Master Slave replication from a single mysql master to a single mysql slave/replica.

In this demo we setup two MySQL instances (e.g. mysql-master-1 and mysql-master-2), both working as master. mysql-master-1 replicates data from mysql-master-2 and mysql-master-2 replicates data from mysql-master-1.

### !!! Prerequisite : You would need Docker installed on your system to follow along this demo.

We would run both mysql-master-1 and mysql-master-2 as docker containers, using docker compose.



