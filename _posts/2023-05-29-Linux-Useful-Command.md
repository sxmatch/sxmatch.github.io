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