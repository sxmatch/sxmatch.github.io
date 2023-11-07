---
layout: post
category: Database
tagline: "keep simple"
tags: [Database, OpenStack]
---
{% include JB/setup %}

**If need to reprint, please indicate the source**

**Copyright: 王昊 Hao Wang @sxmatch**

*2023/11/01*

-------
---

# Install and Config PostgreSQL Cluster

1. Create VMs for DB cluster

2. Config the source repo of PostgreSQL:
   
   ```shell
   # Add exclude=postgresql* to [baseos] section in CentOS-Stream-BaseOS.repo
   # otherwise dependencies might resolve to the postgresql supplied by the base repository
   
   # Install the repository RPM:
   sudo dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm
   
   # Disable the built-in PostgreSQL module:
   sudo dnf -qy module disable postgresql
   ```

3. Install PostgreSQL with yum on every node of DB cluster:
   
   ```shell
   # Install PostgreSQL:
   sudo dnf install -y postgresql15-server
   
   # Optionally initialize the database and enable automatic start:
   sudo /usr/pgsql-15/bin/postgresql-15-setup initdb
   sudo systemctl enable postgresql-15
   sudo systemctl start postgresql-15
   ```

4. Config the password for user 'postgres'
   
   ```shell
   sudo passwd postgres
   su - postgres
   psql
   \l
   ALTER USER postgres PASSWORD 'postgres';
   \q
   ```

5. Install the replication controller repmgr on every nodes of DB cluster.
   
   ```shell
   sudo dnf install -y repmgr_15
   ```

6. PostgreSQL configuration for replication
   
   ```shell
   # On the primary server, a PostgreSQL instance must be initialised and running
   touch /var/lib/pgsql/15/data/postgresql.replication.conf
   # Edit this file, add those configurations
   
   # Enable replication connections; set this value to at least one more
   # than the number of standbys which will connect to this server
   # (note that repmgr will execute "pg_basebackup" in WAL streaming mode,
   # which requires two free WAL senders).
   #
   # See: https://www.postgresql.org/docs/current/runtime-config-replication.html#GUC-MAX-WAL-SENDERS
   max_wal_senders = 10
   
   # If using replication slots, set this value to at least one more
   # than the number of standbys which will connect to this server.
   # Note that repmgr will only make use of replication slots if
   # "use_replication_slots" is set to "true" in "repmgr.conf".
   # (If you are not intending to use replication slots, this value
   # can be set to "0").
   #
   # See: https://www.postgresql.org/docs/current/runtime-config-replication.html#GUC-MAX-REPLICATION-SLOTS
   max_replication_slots = 10
   
   # Ensure WAL files contain enough information to enable read-only queries
   # on the standby.
   #
   #  PostgreSQL 9.5 and earlier: one of 'hot_standby' or 'logical'
   #  PostgreSQL 9.6 and later: one of 'replica' or 'logical'
   #    ('hot_standby' will still be accepted as an alias for 'replica')
   #
   # See: https://www.postgresql.org/docs/current/runtime-config-wal.html#GUC-WAL-LEVEL
   wal_level = 'hot_standby'
   
   # Enable read-only queries on a standby
   # (Note: this will be ignored on a primary but we recommend including
   # it anyway, in case the primary later becomes a standby)
   #
   # See: https://www.postgresql.org/docs/current/runtime-config-replication.html#GUC-HOT-STANDBY
   hot_standby = on
   
   # Enable WAL file archiving
   #
   # See: https://www.postgresql.org/docs/current/runtime-config-wal.html#GUC-ARCHIVE-MODE
   archive_mode = on
   
   # Set archive command to a dummy command; this can later be changed without
   # needing to restart the PostgreSQL instance.
   #
   # See: https://www.postgresql.org/docs/current/runtime-config-wal.html#GUC-ARCHIVE-COMMAND
   archive_command = '/bin/true'
   
   # Edit /var/lib/pgsql/15/data/postgresql.conf to add the replication conf file above
   include = 'postgresql.replication.conf'
   
   # Change listen address to all in /var/lib/pgsql/15/data/postgresql.conf
   listen_addresses = '*'
   ```

7. Create a dedicated PostgreSQL superuser account and a database for the repmgr metadata
   
   ```shell
   su - postgres
   sudo -u postgres createuser -s repmgr
   sudo -u postgres createdb repmgr -O repmgr
   psql
   ALTER USER repmgr SET search_path TO repmgr, "$user", public;
   
   #ADD user permission in /var/lib/pgsql/15/data/pg_hba.conf
   host    replication     repmgr          192.168.70.0/24         trust
   host    repmgr          repmgr          192.168.70.0/24         trust
   systemctl restart postgresql-15
   ```

8. Edit the `repmgr.conf` file on the primary server under /etc/repmgr/15. The file must contain at least the following parameters:
   
   ```shell
   node_id=1
   node_name='PG_DB_CLUSTER_1'
   conninfo='host=PG_DB_CLUSTER_1 user=repmgr dbname=repmgr connect_timeout=2'
   data_directory='/var/lib/pgsql/15/data'
   ```

9. **Add the all the cluster's IP address and hostname in /etc/hosts on every node.**

10. To enable repmgr to support a replication cluster, the primary node must be registered with repmgr.
    
    ```shell
    su - postgres
    /usr/pgsql-15/bin/repmgr -f /etc/repmgr/15/repmgr.conf primary register
    ```

11. Verify status of the cluster like this:
    
    ```shell
    su - postgres
    /usr/pgsql-15/bin/repmgr -f /etc/repmgr/15/repmgr.conf cluster show
    
    # Should show like this:
     ID | Name            | Role    | Status    | Upstream | Location | Priority | Timeline | Connection string
    ----+-----------------+---------+-----------+----------+----------+----------+----------+-----------------------------------------------------------------
     1  | PG_DB_CLUSTER_1 | primary | * running |          | default  | 100      | 1        | host=PG_DB_CLUSTER_1 user=repmgr dbname=repmgr connect_timeout=2
    ```

12. **Export the primary DB vm into an image**

13. Prepare the standby DB vm, add the repo of PostgreSQL, install the package and do *not* create a PostgreSQL instance(i.e. do not execute initdb or any database creation scripts provided by packages)
    
    ```shell
    sudo dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm
    sudo dnf -qy module disable postgresql
    sudo dnf install -y postgresql15-server
    sudo systemctl enable postgresql-15
    sudo dnf install -y repmgr_15
    ```

14. Check the primary database is reachable from the standby using psql
    
    ```shell
    psql 'host=<primary node name: PG_DB_CLUSTER_1> user=repmgr dbname=repmgr connect_timeout=2'
    ```

15. Create a `repmgr.conf` file on the standby server. It must contain at least the same parameters as the primary's `repmgr.conf`, but with the mandatory values `node`, `node_name`, `conninfo` (and possibly `data_directory`) adjusted accordingly:
    
    ```shell
    node_id=2
    node_name='PG_DB_CLUSTER_2'
    conninfo='host=PG_DB_CLUSTER_2 user=repmgr dbname=repmgr connect_timeout=2'
    data_directory='/var/lib/pgsql/15/data'
    ```

16. Use the `--dry-run` option to check the standby can be cloned:
    
    ```shell
    su - postgres
    /usr/pgsql-15/bin/repmgr -h PG_DB_CLUSTER_1 -U repmgr -d repmgr -f /etc/repmgr/15/repmgr.conf standby clone --dry-run
    
    # Should seem like this:
    NOTICE: destination directory "/var/lib/pgsql/15/data" provided
    INFO: connecting to source node
    DETAIL: connection string is: host=PG_DB_CLUSTER_1 user=repmgr dbname=repmgr
    DETAIL: current installation size is 30 MB
    INFO: "repmgr" extension is installed in database "repmgr"
    INFO: replication slot usage not requested;  no replication slot will be set up for this standby
    INFO: parameter "max_wal_senders" set to 10
    NOTICE: checking for available walsenders on the source node (2 required)
    INFO: sufficient walsenders available on the source node
    DETAIL: 2 required, 10 available
    NOTICE: checking replication connections can be made to the source server (2 required)
    INFO: required number of replication connections could be made to the source server
    DETAIL: 2 replication connections required
    WARNING: data checksums are not enabled and "wal_log_hints" is "off"
    DETAIL: pg_rewind requires "wal_log_hints" to be enabled
    NOTICE: standby will attach to upstream node 1
    HINT: consider using the -c/--fast-checkpoint option
    INFO: would execute:
      /usr/pgsql-15/bin/pg_basebackup -l "repmgr base backup"  -D /var/lib/pgsql/15/data -h PG_DB_CLUSTER_1 -p 5432 -U repmgr -X stream
    INFO: all prerequisites for "standby clone" are met
    ```

17. **Export the standby DB node into an image**.

18. If no problems are reported, the standby can then be cloned with:
    
    ```shell
    su - postgres
    /usr/pgsql-15/bin/repmgr -h PG_DB_CLUSTER_1 -U repmgr -d repmgr -f /etc/repmgr/15/repmgr.conf standby clone
    # By default, any configuration files in the primary's data directory will be copied to the standby.
    # Typically these will be postgresql.conf, postgresql.auto.conf, pg_hba.conf and pg_ident.conf.
    # These may require modification before the standby is started 
    # Make any adjustments to the standby's PostgreSQL configuration files now, then start the server
    sudo systemctl start postgresql-15
    ```

19. Verify replication is functioning:
    
    ```shell
    # Connect to the primary server and execute
    psql 'host=PG_DB_CLUSTER_1 user=repmgr dbname=repmgr connect_timeout=2'
    SELECT * FROM pg_stat_replication;
    # This shows that the previously cloned standby (PG_DB_CLUSTER_2 shown in the field application_name) has connected to the primary from IP address.
    ```

20. Register the standby server on standby node with:
    
    ```shell
    su - postgres
    /usr/pgsql-15/bin/repmgr -f /etc/repmgr/15/repmgr.conf standby register
    ```

21. Check the node is registered by executing `repmgr cluster show` on the standby:
    
    ```shell
    su - postgres
    /usr/pgsql-15/bin/repmgr -f /etc/repmgr/15/repmgr.conf cluster show
    ```

22. Both nodes are now registered with repmgr and the records have been copied to the standby server.
