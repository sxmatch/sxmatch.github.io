---
layout: post
category : Linux
tagline: "keep simple"
tags : [Linux, Command]
---
{% include JB/setup %}

**If need to reprint, please indicate the source**

**Copyright: 王昊 Hao Wang @sxmatch**

*2023/5/29*

-------
---

# Linux Useful Commands

1. Download directory & subdirectories by Wget
   ```wget -r -np -nH --cut-dirs=1 -R index.* {download_url} -P {save to directory}```
   If don't want the directories on the download path, we can use -nH and --cut-dirs to control this.

2. Add IP address in Centos/Rhel
   ```ip addr add 192.168.100.145/24 dev eth1:1```

3. Delete route in Centos/Rhel
   ```ip route del default```

4. Send Http Request by curl
   
   ```shell
   curl -g -i -X POST https://<openstack service public endpoint>:8776/v3/4abbbf3234ce48f092ff7fd1f2392f84/manageable_volumes
      -H "Accept: application/json" -H "Content-Type: application/json" 
      -H "X-Auth-Token: gAAAAABkkPpmt3j0vy4UMsqaOR-URDGmeAXuz1figk2JSiI-SIIkOsKUGTd4Zw36IwFDyOtAwnHAJte4nS3mCV6nTHUWPxj3fUqmC86HF3rhPQ0rB5HEnAgh8vQjbDBfrll66rSoaKR9-xnVoAF0k74E0iwDof9-eCimSfEBXU8Abv5_2hjfsWc"
      -H "OpenStack-API-Version: volume 3.66"
      -d '{"volume": {"host": "os-control-1@rbd-ssd#rbd-ssd", "ref": {"source-name": "test_hw"},
      "name": "managed_volume_hw", "availability_zone": "nova", "description": "Volume imported from existingLV", "volume_type": "gold", "bootable": true}}'
   ```

5. Add route in Centos/Rhel
   ```ip route add default via 192.168.1.1 dev em1```

6. Set the user data to config the default route
   
   ```shell
   net_path = "/etc/sysconfig/network-scripts/ifcfg-eth0"
   script = f"#!/bin/bash\necho 'DEFROUTE=no' >> {net_path}"
   script += "\n ip route del default"
   user_data = base64.b64encode(script.encode("utf-8")).decode('utf-8')
   server["user_data"] = user_data
   ```

7. Get the Storage Usage of Ceph Cluster
   
   ```shell
   Ceph df detail
   Ceph osd df tree
   ```

8. systemctl reset-failed tftp@tftp.service  to clean the not-found failed services.

9. To clear the alert of ceph "xxx daemons have recently crashed",  use ```ceph crash ls & ceph crash archive-all```

10. git fetch origin --prune

11. rabbitmqctl cluster_status rabbitmqctl list_queues type

12. Get all children of rbd image snapshot(include the trash): rbd children -a image/865e26cc-c520-4588-9bbe-5d0c342951d8@snap

13. Remove the image from the trash: rbd --pool volume-ssd trash remove 5a98c5f62bc3d8

14. Restore the image from the trash: rbd trash restore 62c12d9c849566 --pool volume-ssd

15. Unprotect the snapshot: rbd snap unprotect --pool volume-ssd volume-a9f01c5f-6df7-4363-89d3-c9e0b4f28601@__snap_for_clone

16. Remove the snapshot: rbd snap remove --pool volume-ssd volume-a9f01c5f-6df7-4363-89d3-c9e0b4f28601@__snap_for_clone

17. Remove the volume: rbd remove --pool volume-ssd volume-a9f01c5f-6df7-4363-89d3-c9e0b4f28601
    
    ```shell
    curl -g -i -X POST http://172.19.25.18/volume/v3/f44b903587ac4310af9ae37feaff73d0/volumes/fbf408de-5773-48cd-84d8-446faf223d71/action -H "Accept: application/json" -H "Content-Type: application/json" -H "X-Auth-Token: gAAAAABk97Yby6yjZIox9TVAIxsoBIrpH_-oTA65_SwDiQS8qmQOPX1HEggNtKCvLWsuT3IXO7HdAOg__oC0atjZ4VAKzT0hDjeer7XE40sZvgdwZ9L4gEl3KatceOEUd4bDIm8CuKGgKpjGPkPmmTg2Reg-D5bawReNuYDl5EkWGdXZCttZRNE" -H "X-Service-Token: gAAAAABk97Yby6yjZIox9TVAIxsoBIrpH_-oTA65_SwDiQS8qmQOPX1HEggNtKCvLWsuT3IXO7HdAOg__oC0atjZ4VAKzT0hDjeer7XE40sZvgdwZ9L4gEl3KatceOEUd4bDIm8CuKGgKpjGPkPmmTg2Reg-D5bawReNuYDl5EkWGdXZCttZRNE" -H "OpenStack-API-Version: volume 3.66" -d '{"os-detach": {"attachment_id": "49930aa2-6564-461f-9af8-564cfecd56cb"}}'curl -g -i -X POST http://172.19.25.18/volume/v3/f44b903587ac4310af9ae37feaff73d0/volumes/fbf408de-5773-48cd-84d8-446faf223d71/action -H "Accept: application/json" -H "Content-Type: application/json" -H "X-Auth-Token: gAAAAABk97Yby6yjZIox9TVAIxsoBIrpH_-oTA65_SwDiQS8qmQOPX1HEggNtKCvLWsuT3IXO7HdAOg__oC0atjZ4VAKzT0hDjeer7XE40sZvgdwZ9L4gEl3KatceOEUd4bDIm8CuKGgKpjGPkPmmTg2Reg-D5bawReNuYDl5EkWGdXZCttZRNE" -H "X-Service-Token: gAAAAABk97Yby6yjZIox9TVAIxsoBIrpH_-oTA65_SwDiQS8qmQOPX1HEggNtKCvLWsuT3IXO7HdAOg__oC0atjZ4VAKzT0hDjeer7XE40sZvgdwZ9L4gEl3KatceOEUd4bDIm8CuKGgKpjGPkPmmTg2Reg-D5bawReNuYDl5EkWGdXZCttZRNE" -H "OpenStack-API-Version: volume 3.66" -d '{"os-detach": {"attachment_id": "49930aa2-6564-461f-9af8-564cfecd56cb"}}'
    ```

18. Run the docker container:
    
    ```shell
    docker run -d --name monitor --net host -v /etc/localtime:/etc/localtime:ro -v /etc/cms:/etc/cms:ro -v /var/log/cms:/var/log/cms:z -v /usr/local/cms:/usr/local/cms:z -v /etc/pki/ca-trust/source/anchors:/etc/pki/ca-trust/source/anchors:ro -v /root/.ssh:/root/.ssh:ro -e PYTHONPATH=/usr/local/cms -w /usr/local/cms/monitor registry.image.com/cms:1.0 /usr/bin/bash -c "update-ca-trust && python3 /usr/local/cms/monitor/server.py"
    ```

19. Resize and extend Linux filesystem
    
    ```shell
    # convert MBR to GPT
    lsblk
    gdisk /dev/vda
    W
    Y
    sudo gdisk /dev/vda
    N
    34
    \n
    ef02
    W
    Y
    Y
    sudo partprobe
    grub2-install /dev/vda
    sudo grub2-mkconfig -o /boot/grub2/grub.cfg
    sudo parted /dev/vda print
    
    #extend the partitions in linux
    gdisk /dev/vda
    p
    d
    n
    \n
    \n
    w
    y
    sudo partprobe
    
    growpart /dev/vda 1
    xfs_growfs /dev/vda1
    ```

20. Cloud-Init for user password
    
    ```shell
    #cloud-config
    users:
    - name: root
      lock_passwd: false
      hashed_passwd: '$1$SaltSalt$7djuPQtXUUgTpJFkzZrCk0'
    
    #cloud-config
    users:
    - name: root
      lock_passwd: false
      hashed_passwd: '$1$SaltSalt$toXqleTqTdz/t/WB5JNTz.'
    disable_root: false
    ssh_pwauth: true
    
    runcmd:
    - sed -i -e 's/^#\?PermitRootLogin.*/PermitRootLogin yes/' /etc/ssh/sshd_config
    - systemctl restart sshd
    ```

21. Sync the repo
    
    ```shell
    dnf reposync --repo zabbix --download-metadata --newest-only
    dnf install createrepo
    createrepo /opt/depot/grafana.20231116/CentOS-Stream-9/
    dnf config-manager --add-repo /opt/depot/grafana.20231116/CentOS-Stream-9
    ```

22. Improve Ceph recovery speed
    
    ```shell
    1.set “osd_mclock_cost_per_byte_usec_hdd” to 0.1
    2.set "osd_mclock_max_capacity_iops_hdd" to 10000
    3.finally soluation: upgrade to 17.2.7
    ```

23. Select all in VI
    
    ```shell
    gg"+yG
    ```

24. Check the state of replication in PGSQL
    
    ```shell
    select usename, application_name, client_addr, state, sync_priority, sync_state from pg_stat_replication;
    systemctl stop postgresql
    rm -rf /var/lib/pgsql/data/*
    su - postgres
    pg_basebackup -h 192.168.20.21 -D /var/lib/pgsql/data -p 5432 -X stream -U replica -w
    su - postgres
    touch /var/lib/pgsql/data/standby.signal
    chmod 600 /var/lib/pgsql/data/standby.signal
    systemctl start postgresql
    ```

25. Deployment for rsyslogd
    
    ```shell
    dnf install -y rsyslog-elasticsearch
    mkdir /var/log/rsyslog
    vi /etc/rsyslog.d/postgresql.conf
    setsebool -P logging_syslogd_list_non_security_dirs 1
    systemctl restart rsyslog
    
    sudo semanage fcontext -a -t syslogd_var_lib_t "/var/lib/pgsql/data/log/*"
    sudo restorecon -R -v /var/lib/pgsql/data/log
    
    sudo semanage fcontext -d -t syslogd_var_lib_t "/var/lib/pgsql/*"
    sudo semanage fcontext -a -t postgresql_db_t "/var/lib/pgsql/*"
    sudo restorecon -R -v /var/lib/pgsql
    ```

26. Configuration of rsyslogd
    
    ```shell
    module(load="imfile")
    module(load="omelasticsearch")
    
    input(type="imfile"
      file="/var/lib/pgsql/data/log/postgresql-Mon.log"
      tag="fio.postgresql.Mon"
      startmsg.regex="^[0-9]{4}-[0-9]{2}-[0-9]{2} [0-9]{2}:[0-9]{2}:[0-9]{2}(.[0-9]+)? (UTC) "
      reopenOnTruncate="on"
    )
    
    input(type="imfile"
      file="/var/lib/pgsql/data/log/postgresql-Tue.log"
      tag="fio.postgresql.Tue"
      startmsg.regex="^[0-9]{4}-[0-9]{2}-[0-9]{2} [0-9]{2}:[0-9]{2}:[0-9]{2}(.[0-9]+)? (UTC) "
      reopenOnTruncate="on"
    )
    
    input(type="imfile"
     file="/var/lib/pgsql/data/log/postgresql-Wed.log"
     tag="fio.postgresql.Wed"
     startmsg.regex="^[0-9]{4}-[0-9]{2}-[0-9]{2} [0-9]{2}:[0-9]{2}:[0-9]{2}(.[0-9]+)? (UTC) "
     reopenOnTruncate="on"
    )
    
    input(type="imfile"
     file="/var/lib/pgsql/data/log/postgresql-Thu.log"
     tag="fio.postgresql.Thu"
     startmsg.regex="^[0-9]{4}-[0-9]{2}-[0-9]{2} [0-9]{2}:[0-9]{2}:[0-9]{2}(.[0-9]+)? (UTC) "
     reopenOnTruncate="on"
    )
    
    input(type="imfile"
     file="/var/lib/pgsql/data/log/postgresql-Fri.log"
     tag="fio.postgresql.Fri"
     startmsg.regex="^[0-9]{4}-[0-9]{2}-[0-9]{2} [0-9]{2}:[0-9]{2}:[0-9]{2}(.[0-9]+)? (UTC) "
     reopenOnTruncate="on"
    )
    
    input(type="imfile"
     file="/var/lib/pgsql/data/log/postgresql-Sat.log"
     tag="fio.postgresql.Sat"
     startmsg.regex="^[0-9]{4}-[0-9]{2}-[0-9]{2} [0-9]{2}:[0-9]{2}:[0-9]{2}(.[0-9]+)? (UTC) "
     reopenOnTruncate="on"
    )
    
    input(type="imfile"
     file="/var/lib/pgsql/data/log/postgresql-Sun.log"
     tag="fio.postgresql.Sun"
     startmsg.regex="^[0-9]{4}-[0-9]{2}-[0-9]{2} [0-9]{2}:[0-9]{2}:[0-9]{2}(.[0-9]+)? (UTC) "
     reopenOnTruncate="on"
    )
    
    template (name="rsyslog-node-index" type="list"
    )
    {
     constant(value="postgresql-xfa-qa-pg-cluster-1-3-" )
     property(dateFormat="year" name="timereported" )
     constant(value="." )
     property(dateFormat="month" name="timereported" )
     constant(value="." )
     property(dateFormat="week" name="timereported" )
    }
    
    template (name="rsyslog-record" type="list"
     option.jsonf="on")
    {
     property(dateFormat="rfc3339" format="jsonf" name="timereported" outname="@timestamp" )
     property(format="jsonf" name="hostname" outname="host" )
     property(format="jsonf" name="syslogseverity" outname="severity" )
     property(format="jsonf" name="syslogfacility-text" outname="facility" )
     property(format="jsonf" name="syslogtag" outname="tag" )
     property(format="jsonf" name="app-name" outname="source" )
     property(format="jsonf" name="msg" outname="message" )
     property(format="jsonf" name="$!metadata!filename" outname="file" )
     constant(format="jsonf" outname="cloud" value="os.vancouver-a.corp.fortinet.com" )
     constant(format="jsonf" outname="region" value="regionOne" )
    }
    
    #elasticsearch
    action(type="omelasticsearch"
    name="elasticsearch"
    allowunsignedcerts="on"
    bulkmode="on"
    dynSearchIndex="on"
    action.errorfile="/var/log/rsyslog/omelasticsearch.log"
    action.errorfile.maxsize="1000000000"
    pwd="sendlog"
    searchIndex="rsyslog-node-index"
    server=["192.168.200.172:9200"]
    skipverifyhost="on"
    template="rsyslog-record"
    uid="sender"
    searchType="_doc"
    ) 
    ```

27. TCP Dump Command
    
    ```shell
    tcpdump -ei eth1 port 3128
    
    nmcli connection show
    nmcli connection modify "Wired connection 1" ipv4.gateway ""
    nmcli connection modify "Wired connection 1" ipv4.never-default true
    nmcli connection down "Wired connection 1"
    nmcli connection up "Wired connection 1"
    
    echo "1 rt1" | sudo tee -a /etc/iproute2/rt_tables
    sudo ip route add 192.168.30.0/24 dev eth1 src 192.168.30.2 table rt1
    sudo ip route add default via 192.168.30.1 dev eth1 table rt1
    ip rule add from 192.168.30.173/32 table rt1
    
    echo "1 rt1" |  tee -a /etc/iproute2/rt_tables
    echo "192.168.30.0/24 dev eth1 src 192.168.30.10 table rt1" | sudo tee /etc/sysconfig/network-scripts/route-eth1
    echo "default via 192.168.30.1 dev eth1 table rt1" >> /etc/sysconfig/network-scripts/route-eth1
    echo "from 192.168.30.10/32 table rt1" | sudo tee /etc/sysconfig/network-scripts/rule-eth1
    
    echo "1 rt1" |  tee -a /etc/iproute2/rt_tables
    nmcli connection show
    nmcli connection modify "System eth1" +ipv4.route-table 1
    nmcli connection modify "System eth1" +ipv4.routes "192.168.30.0/24 192.168.30.1 1"
    ip rule list
    nmcli connection modify "System eth1" +ipv4.routing-rules "priority 32765 from 192.168.30.173/32 lookup 1"
    nmcli connection modify "System eth1" ipv4.never-default false
    nmcli connection down "System eth1"
    nmcli connection up "System eth1"
    ```

28. Add firewall rule
    
    ```shell
    sudo firewall-cmd --zone=public --add-port=443/tcp --permanent
    sudo firewall-cmd --zone=public --add-service=https --permanent
    sudo firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source address="0.0.0.0/0" accept' --permanent
    sudo firewall-cmd --reload
    sudo nft list ruleset
    sudo nft flush ruleset
    ```