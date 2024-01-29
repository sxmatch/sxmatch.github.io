# Uncontainered Zabbix-Proxy

## Install Zabbix-Proxy with RPM

1. Install Zabbix repository
   
   ```shell
   rpm -Uvh https://repo.zabbix.com/zabbix/6.4/rhel/9/x86_64/zabbix-release-6.4-1.el9.noarch.rpm
   dnf clean all
   ```

2. Install zabbix-proxy with specific database type
   
   ```shell
   dnf install -y zabbix-proxy-mysql zabbix-sql-scripts zabbix-selinux-policy
   ```

## Install MariaDB with RPM

1. Install MariaDB and Galera packages
   
   ```shell
   dnf install -y galera mariadb-server mariadb-server-galera
   systemctl enable --now mariadb
   mysql_secure_installation
   ```

2. Creating proxy database
   
   ```shell
   shell> mysql -uroot -pRoot1234
   mysql> create database zabbix_proxy character set utf8mb4 collate utf8mb4_bin;
   mysql> create user zabbix@localhost identified by 'Zabbix1234';
   mysql> grant all privileges on zabbix_proxy.* to zabbix@localhost;
   mysql> set global log_bin_trust_function_creators = 1;
   mysql> quit;
   ```

3. Then import the initial schema. Make sure to insert correct version.
   
   ```shell
   cat /usr/share/zabbix-sql-scripts/mysql/proxy.sql | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix_proxy
   ```

4. Disable log_bin_trust_function_creators option after importing database schema.
   
   ```shell
   shell> mysql -uroot -pRoot1234
   mysql> set global log_bin_trust_function_creators = 0;
   mysql> quit;
   ```

5. Configure the database for Zabbix proxy in /etc/zabbix/zabbix_proxy.conf
   
   ```shell
   DBPassword=Zabbix1234
   Hostname=zabbix-proxy.poc
   Server=zabbix.com
   ConfigFrequency=600
   CacheSize=512M
   DBHost=127.0.0.1
   DBPort=3306
   AllowRoot=1
   
   SSLCertLocation=/var/lib/zabbix/ssl/certs/
   SSLKeyLocation=/var/lib/zabbix/ssl/keys/
   SSLCALocation=/var/lib/zabbix/ssl/ssl_ca/
   ```

## Start zabbix_proxy

1. Start Zabbix Proxy
   
   ```shell
   systemctl enable zabbix-proxy
   systemctl restart zabbix-proxy
   ```

2. Copy script
   
   ```shell
   cp ceph /usr/lib/zabbix/externalscripts/
   cp get-io /usr/lib/zabbix/externalscripts/
   cp openstack /usr/lib/zabbix/externalscripts/
   ```

3. Config logrotate of zabbix and maradb
   
   ```shell
   dnf install -y logrotate
   cat > /etc/logrotate.d/zabbix << __EOF__
   /var/log/zabbix/*.log {
       rotate 7
       daily
       compress
       missingok
       notifempty
   }
   __EOF__
   mysql mysql -uroot -pRoot1234 \
   -e "set global binlog_expire_logs_seconds=60*60*24; \
       show variables like 'binlog_expire_logs_seconds';"
   ```
