# Simple grafana monitoring of MySQL metrics

> This repo attempts to demonstrate and provide a usable mysql install on docker, with grafana monitoring captured by prometheus

# Getting started

## Pre-req's 

Before getting started, ensure that you have the following installed locally:

- Vagrant
- Virtualbox

## Starting site

Simply run the following in this directory:

```bash
vagrant up
vagrant status
docker compose up -d

docker ps
CONTAINER ID        IMAGE                         COMMAND                  CREATED              STATUS              PORTS                                NAMES
f85a5e1060f6        prom/mysqld-exporter:latest   "/bin/mysqld_exporter"   About a minute ago   Up 9 seconds        0.0.0.0:9104->9104/tcp               mysql_exporter8
e89ddb1b090e        prom/prometheus               "/bin/prometheus --c…"   15 minutes ago       Up 15 minutes       0.0.0.0:9090->9090/tcp               prometheus8
b15668695d0f        grafana/grafana               "/run.sh"                16 minutes ago       Up 16 minutes       0.0.0.0:3000->3000/tcp               grafana8
3c0f88cffc07        mysql:latest                  "docker-entrypoint.s…"   15 minutes ago       Up 10 minutes       33060/tcp, 0.0.0.0:53306->3306/tcp   mysql8

1. run MySQLd exporter http://0.0.0.0:9104
2. see MySQLd exporter
4. see metrics
5. run prometheus http://0.0.0.0:9090
7. click status
8. click targets
9. verify all up
10. run grafana http://0.0.0.0:3000
1. add datasource prometheus
2. set name prometheus
3. set url=http://prometheus:9090/
4. connect and add all 3 dashboards
5. add datasource mysql
6. set name mysql
7. set Host: 172.26.0.2:3306
8. set database: testdb
9. set User: root
10. set Pass: 123
11. connect
12. if u got 'this authentication plugin is not supported'
13. docker exec -it --user root mysql8 bash
14.  mysql --user=root --password=123
15.  alter user 'root' identified with mysql_native_password by '123';
16.  exit
17. try connecting again in grafana
```

# MySQL 

Simple install that can be accessed via:

```
mysql.server start
Host: 192.168.33.10
Port: 3306
User: root
Pass: 123
```
#### mysql problem shooting 

```
docker ps
docker logs mysql8
dokcer rm mysql8 -f
docker run --name mysql8 -d -e MYSQL_ROOT_PASSWORD=123 -p 53306:3306 mysql:latest
docker inspect mysql8
look for "IPAddress" in json
put the "IPAddress" in grafana
look for #### "Destination": "/var/lib/mysql"

open mysql workbench
Host: 0.0.0.0
Port: 53306
User: root
Pass: 123
run db_schema.sql
run db_data.sql
```
  
# MySQL Metrics

In order to provide this info in a fancy way, I've hooked in grafana and set it up with the percona dashboards to visualize the info.  You can access the info here:

[Grafana Percona dashboards](http://192.168.33.10/dashboard/db/mysql-overview-percona-app?from=now-15m&to=now)

```
Username: admin
Password: admin
```

# How it works

The way that this vagrant instance is set up is by installing docker on the vagrant instance, and then creating a few containers to do all the work.  In this setup we have the following:

1. `mysql` container: simple setup of a mysql instance
1. `prometheus` container: timeseries database storage for metrics storage
1. `grafana` container: graphing tool to visualize information from mysql / prometheus
1. `prom_mysql_exporter` container: tool to collect data from mysql and store in prometheus


# Design decision

This repo is consciously designed to have two separate aspects of the setup (docker + ansible within vagrant), for 2 reasons: the grafana docker image doesn't contain much by way of configuration of the dashboards/users, and, that it should not be used as a production service.  It should be broken down into its separate components based on your desired infrastructure.  This is just a demo to get you up and running quickly. 

 
