---
layout: post
title: "Galera Cluster on Debian 8"
date: 2016-05-05 16:25:06 -0700
comments: true
---

In this stage you need two nodes with Debian OS for deploy Galera Cluster Maria DB. The next entry add other two nodes with HAProxy and VIP for load balancer to Cluster.

The first step is add the repository of mariadb to Debian (Node1 and node2):
<pre><code>sudo apt-get install software-properties-common
sudo apt-key adv --recv-keys --keyserver keyserver.ubuntu.com 0xcbcb082a1bb943db
sudo add-apt-repository 'deb [arch=amd64,i386] http://mariadb.kisiek.net/repo/10.0/debian jessie main'
sudo apt-get update
sudo apt-get upgrade</code></pre>

When is done, install the software for Galera cluster (Node1 and node2):

<pre><code>apt-get install -y rsync galera mariadb-galera-server</code></pre>

The next step is configure the file of configuration of Galera:

For node01:
<pre><code>echo '[mysqld]
# MySQL Configuration
query_cache_size=0
binlog_format=ROW
default-storage-engine=<strong>innodb</strong>
innodb_autoinc_lock_mode=2
query_cache_type=0
bind-address=<strong>0.0.0.0</strong>

# Galera Provider Configuration
wsrep_provider=/usr/lib/galera/libgalera_smm.so
#wsrep_provider_options="gcache.size=32G"

# Galera Cluster Configuration
wsrep_cluster_name="test_cluster"
wsrep_cluster_address="<strong>gcomm://192.168.30.11:4567,192.168.30.12:4567</strong>"

# Galera Synchronization Congifuration
wsrep_sst_method=rsync
#wsrep_sst_auth=user:pass

# Galera Node Configuration
wsrep_node_address="<strong>192.168.30.11</strong>"
wsrep_node_name="<strong>node01</strong>"' > /etc/mysql/conf.d/galera.cnf ; chmod 770 /etc/mysql/conf.d/galera.cnf</code></pre>

For node02:
<pre><code>echo '[mysqld]
# MySQL Configuration
query_cache_size=0
binlog_format=ROW
default-storage-engine=<strong>innodb</strong>
innodb_autoinc_lock_mode=2
query_cache_type=0
bind-address=<strong>0.0.0.0</strong>

# Galera Provider Configuration
wsrep_provider=/usr/lib/galera/libgalera_smm.so
#wsrep_provider_options="gcache.size=32G"

# Galera Cluster Configuration
wsrep_cluster_name="test_cluster"
wsrep_cluster_address="<strong>gcomm://192.168.30.11:4567,192.168.30.12:4567</strong>"

# Galera Synchronization Congifuration
wsrep_sst_method=rsync
#wsrep_sst_auth=user:pass

# Galera Node Configuration
wsrep_node_address="<strong>192.168.30.12</strong>"
wsrep_node_name="<strong>node02</strong>"' > /etc/mysql/conf.d/galera.cnf ; chmod 770 /etc/mysql/conf.d/galera.cnf</code></pre>


## BONUS configuration

<pre><code>[mysql_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid</code></pre>



Configure the file /etc/hosts (Node1 and node2):
<pre><code>192.168.30.11 node01
192.168.30.12 node02</code></pre>

Copy file /etc/mysql/debian.cnf to the node01 to node02

Stop the service MySQL (Node1 and node2):
<pre><code>service mysql stop</code></pre>

Execute the next command for create new cluster (Node1):
<pre><code>service mysql start --wsrep-new-cluster</code></pre>

<strong>ERROR:</strong> WSREP: gcs connect failed: Connection timed out
<strong>SOLUTION:</strong> 
Execute:
<pre><code>service mysql bootstrap</code></pre>

Restart all services of MySQL and Galera (Node1 and node2).

The next query response the number of nodes of cluster Galera:
<pre><code>mysql -u root -e 'SELECT VARIABLE_VALUE as "cluster size" FROM INFORMATION_SCHEMA.GLOBAL_STATUS WHERE VARIABLE_NAME="wsrep_cluster_size"' -p</code></pre>

Finish the first part.