# ProxySQL

ProxySQL is an open-source MySQL proxy server, meaning it serves as load balancer among the database and applications that access its database. ProxySQL proven that it can improve the performance by distributing traffic among the pool of multiple of database servers and improve the availability by automatically failing over to a standby if one or more of the database servers fail.

Steps to install ProxySQL

ProxySQL able to be installed in two ways one is on the Docker and next is you able to install it on a server and integrate it with the databases that inside the galera cluster.

In this tutorial, the steps that going to be presented is on installing ProxySQL in the docker container and create a load balancer between the applications and databases.

1. Before initialize installation on the ProxySQL installation, there few things needed to do, which is needed to add user which are admin and monitor in the databases inside the galera cluster.

# Create user monitor for ProxySQL connect with the database

CREATE OR REPLACE USER 'monitor'@'%' IDENTIFIED BY 'monitor';
