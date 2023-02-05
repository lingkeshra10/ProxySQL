# ProxySQL

ProxySQL is an open-source MySQL proxy server, meaning it serves as load balancer among the database and applications that access its database. ProxySQL proven that it can improve the performance by distributing traffic among the pool of multiple of database servers and improve the availability by automatically failing over to a standby if one or more of the database servers fail.

Steps to install ProxySQL

ProxySQL able to be installed in two ways one is on the Docker and next is you able to install it on a server and integrate it with the databases that inside the galera cluster.

In this tutorial, the steps that going to be presented is on installing ProxySQL in the docker container and create a load balancer between the applications and databases.

1. Before initialize installation on the ProxySQL installation, there few things needed to do, which is needed to add user which are admin and monitor in the databases inside the galera cluster.

###### Create monitor user for ProxySQL connect with the database

```
mariadb[none]> CREATE OR REPLACE USER 'monitor'@'%' IDENTIFIED BY 'monitor';
mariadb[none]> GRANT SELECT on sys.* to 'monitor'@'%';
mariadb[none]> FLUSH PRIVILEGES;
```

###### Create admin user for application to connect

```
mariadb[none]> CREATE USER 'centagateadmin'@'%' IDENTIFIED BY 'foo123';
mariadb[none]> GRANT ALL ON RECIPES.* TO 'centagateadmin'@'%';
mariadb[none]> GRANT ALL PRIVILEGES ON *.* TO 'centagateadmin'@'%' WITH GRANT OPTION;
mariadb[none]> FLUSH PRIVILEGES;
```

2. After adding user successfully inside the database and next we proceed to setup the proxysql.cnf

3. Prepare the location to setup the proxysql.cnf to intialize the ProxySQL service in the docker. In this tutorial we will use the **/opt** to prepare the ProxySQL config file.

4. Below are the content of the ProxySQL configuration.


