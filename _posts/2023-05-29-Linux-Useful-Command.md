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
   
   ```
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
   
   ```
   net_path = "/etc/sysconfig/network-scripts/ifcfg-eth0"
   script = f"#!/bin/bash\necho 'DEFROUTE=no' >> {net_path}"
   script += "\n ip route del default"
   user_data = base64.b64encode(script.encode("utf-8")).decode('utf-8')
   server["user_data"] = user_data
   ```

7. Get the Storage Usage of Ceph Cluster
   
   ```
   Ceph df detail
   Ceph osd df tree
   ```

8. systemctl reset-failed tftp@tftp.service  to clean the not-found failed services.

9. To clear the alert of ceph "xxx daemons have recently crashed",  use ```ceph crash ls & ceph crash archive-all```

10. git fetch origin --prune

11. rabbitmqctl cluster_status rabbitmqctl list_queues type