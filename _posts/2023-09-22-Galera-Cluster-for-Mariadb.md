---
layout: post
category : Monitoring
tagline: "keep simple"
tags : [OpenStack, Zabbix， Mariadb]
---
{% include JB/setup %}

**If need to reprint, please indicate the source**

**Copyright: 王昊 Hao Wang @sxmatch**

*2023/9/22*

-------
---

# Galera Cluster for Mariadb with Zabbix

1. Export the zabbix database from MySQL
   
   ```shell
   docker exec -it mysql_server bash
   mysqldump -u root -p zabbix > /var/lib/mysql/zabbix.sql
   ```

2. stop and remove the containers of MySQL database

3. dnf install -y galera mariadb-server mariadb-server-galera

4. Enable Mariadb service: systemctl enable --now mariadb

5. Init the Mariadb: mysql_secure_installation

6. Config Galera configuration file
   
   ```shell
   [mysqld]
   basedir = /usr
   bind-address = 192.168.1.2
   port = 3306
   binlog_format = ROW
   default-storage-engine = innodb
   innodb_autoinc_lock_mode = 2
   collation-server = utf8_general_ci
   init-connect = SET NAMES utf8
   character-set-server = utf8
   datadir = /var/lib/mysql/
   wsrep_cluster_address = gcomm://192.168.1.2:4567,192.168.1.3:4567,192.168.1.4:4567
   wsrep_provider_options = gmcast.listen_addr=tcp://192.168.1.2:4567;ist.recv_addr=192.168.1.2:4568;
   wsrep_node_address = 192.168.1.2:4567
   wsrep_sst_receive_address = 192.168.1.2:4444
   wsrep_provider = /usr/lib64/galera/libgalera_smm.so
   wsrep_cluster_name = monitor
   wsrep_node_name = monitor-test-1-1.novalocal
   wsrep_sst_method = rsync
   wsrep_sst_auth = root:root
   wsrep_slave_threads = 4
   wsrep_on = ON
   max_connections = 10000
   key_buffer_size = 64M
   max_heap_table_size = 64M
   tmp_table_size = 64M
   innodb_buffer_pool_size = 8192M
   innodb_lock_schedule_algorithm = FCFS
   max_allowed_packet = 64M
   innodb_lock_wait_timeout = 90
   ```

7. Stop mariadb service: systemctl stop mariadb

8. Start the Galera cluster: galera_new_cluster

9. Create the Zabbix database and user, and import the Zabbix db data.
   
   ```shell
   mysql> create database zabbix character set utf8mb4 collate utf8mb4_bin;
   mysql> create user 'zabbix'@'%' identified by 'zabbix';
   mysql> grant all privileges on zabbix.* to 'zabbix'@'%';
   mysql> create user 'root'@'%' identified by 'root';
   mysql> grant all privileges on zabbix.* to 'root'@'%';
   mysql> ALTER USER 'root'@'localhost' IDENTIFIED VIA mysql_native_password USING PASSWORD('root');
   mysql> grant all privileges on zabbix.* to 'root'@'localhost';
   mysql> SET GLOBAL log_bin_trust_function_creators = 1;
   mysql> FLUSH PRIVILEGES;
   mysql> quit;
   
   mysql -u root -p zabbix < /var/lib/mysql/zabbix.sql
   SHOW GLOBAL STATUS LIKE 'wsrep_%';
   ```

10. Change the zabbix_server.conf: DBHost=<DB Server IP>. Stop and remove the zabbix_server & zabbix_web containers. Change the docker running script and restart it.
    
    ```shell
    echo "INFO: Start Zabbix server..."
    docker run -d --name zabbix_server \
        -e DB_SERVER_HOST="<DB Server IP>" \
        -e MYSQL_DATABASE="zabbix" \
        -e MYSQL_USER="zabbix" \
        -e MYSQL_PASSWORD="zabbix" \
        -e MYSQL_ROOT_PASSWORD="root" \
        -e ZBX_VAlUECACHESIZE="4096M" \
        -e ZBX_CACHESIZE="4096M" \
        -e ZBX_JAVAGATEWAY="127.0.0.1" \
        -v /etc/zabbix:/etc/zabbix \
        -v /var/lib/mysql/:/var/lib/mysql/:Z \
        --net host \
    "image_repo/zabbix/zabbix-server-mysql:centos-6.0.8"
    
    echo "INFO: Start Zabbix web..."
    docker run -d --name zabbix_web \
        -e ZBX_SERVER_HOST="127.0.0.1" \
        -e DB_SERVER_HOST="<DB Server IP>" \
        -e MYSQL_DATABASE="zabbix" \
        -e MYSQL_USER="zabbix" \
        -e MYSQL_PASSWORD="zabbix" \
        -e MYSQL_ROOT_PASSWORD="root" \
        -v /var/lib/mysql/:/var/lib/mysql/:Z \
        --net host \
    "image_repo/zabbix/zabbix-web-nginx-mysql:6.0-centos-latest"
    ```

11. Enable the check-alive script for keepalived: chcon -t keepalived_unconfined_script_exec_t /etc/keepalived/check-alive

12. Config the Zabbix HA: config HAnodename=<hostname>

13. Reset the mariadb password if forgot:
    
    ```shell
    add skip-grant-tables under /etc/my.cnf
    login mariadb without password
    mysql -u root
    FLUSH PRIVILEGES;
    SET PASSWORD FOR 'root'=PASSWORD('XRtmkhdMXriIu2VvZ8T');
    ```
    
    
