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

```

// Config file contents referred to as "/path/to/proxysql.cnf"
datadir="/var/lib/proxysql"

admin_variables=
{
        admin_credentials="admin:admin;radmin:radmin"
        mysql_ifaces="0.0.0.0:6032"
}

mysql_variables=
{
        threads=4
        max_connections=5000
        default_query_delay=0
        default_query_timeout=36000000
        have_compress=true
        poll_timeout=2000
        interfaces="0.0.0.0:6033"
        default_schema="information_schema,centagate"
        stacksize=1048576
        server_version="5.5.30"
        connect_timeout_server=3000
        monitor_username="monitor"
        monitor_password="monitor"
        monitor_history=600000
        monitor_connect_interval=60000
        monitor_ping_interval=10000
        monitor_read_only_interval=1500
        monitor_read_only_timeout=500
        ping_interval_server_msec=120000
        ping_timeout_server=500
        commands_stats=true
        sessions_sort=true
        connect_retries_on_failure=10
}

//Assign hostgroups id and responsibility
mysql_galera_hostgroups =
(
    {
        writer_hostgroup=10
        backup_writer_hostgroup=20
        reader_hostgroup=30
        offline_hostgroup=9999
        max_writers=1
        writer_is_also_reader=1
        max_transactions_behind=30
        active=1
    }
)

//Assign mysql servers that inside the galera cluster
mysql_servers =
(
    { address="database_IP_1" , port=3306 , hostgroup=10, max_connections=3000 },
    { address="database_IP_2" , port=3306 , hostgroup=10, max_connections=3000 },
    { address="database_IP_3" , port=3306 , hostgroup=10, max_connections=3000 }
)

//Define the query routing 
mysql_query_rules =
(
    {
        rule_id=100
        active=1
        match_pattern="^SELECT .* FOR UPDATE"
        destination_hostgroup=10
        apply=1
    },
    {
        rule_id=200
        active=1
        match_pattern="^SELECT .*"
        destination_hostgroup=30
        apply=1
    },
    {
        rule_id=300
        active=1
        match_pattern=".*"
        destination_hostgroup=10
        apply=1
    }
)

//Define the mysql users that will connect to the applications
mysql_users =
(
    { username = "centagateadmin", password = "foo123", default_hostgroup = 10, transaction_persistent = 0, active = 1 }
)

```
