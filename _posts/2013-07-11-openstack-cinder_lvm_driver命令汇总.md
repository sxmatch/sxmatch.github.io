---
layout: post
category : OpenStack
tagline: "keep simple"
tags : [OpenStack, cinder, LVM]
---
{% include JB/setup %}

**If need to reprint, please indicate the source**

**Copyright: 王昊 Hao Wang @sxmatch**

*2013/07/11*

-------
---

Cinder在使用LVM（逻辑卷管理）作为driver时，在suse环境和不安装lvm的ubuntu环境上需要先执行一系列的命令才能正常使用cinder:

1. 创建物理卷(一般选择服务器没有使用的另一块或多块硬盘作为供cinder使用的物理卷)
   ```pvcreate [磁盘设备路径] 例如：pvcreate /dev/sdb 创建完成后可以使用 pvdisplay 命令查看物理卷详情```

2. 创建卷组（cinder默认使用的卷组名称为cinder-volumes）
   ``` vgcreate [卷组名称] [要添加到卷组的分区或者磁盘] 例如: vgcreate cinder-volumes /dev/sdb 创建完成后可以使用 vgdisplay 命令查看卷组详情```

3. 激活卷组(这步完成后，cinder就可以正常使用lvm了)
   ```vgchange -ay [卷组名称] 例如 vgchange -ay cinder-volumes```

4. 如果卷组的空间不够用了，后续可以添加新的物理卷到卷组中

```vgextend [卷组名] [物理卷路径] 例如：vgextend cinder-volumes /dev/sdc 注意添加新的物理卷也要再次激活一下卷组```

5. 从卷组中删除一个物理卷，首先要确保没有逻辑卷使用这个物理卷
   ```vgreduce [卷组名] [物理卷路径] 例如:vgreduce cinder-volumes /dev/sdc```

6. 创建逻辑卷(cinder的lvm driver就是使用这个命令创建一个卷)
   ```lvcreate -L [卷大小MB] -n [卷名称] [卷组名] 例如: lvcreate -L 1000 -n volume1 cinder-volumes 创建完成后还可以查询一下: lvdisplay```

7. 扩展逻辑卷(可以扩展逻辑卷的大小，根据参数不同有两种方式)
   ```lvextend -L [大小] [逻辑卷路径] 例如：1.扩展到12G lvextend -L 12G /dev/cinder-volumes/volume1 ；2.增加12G lvextend -L +12G /dev/cinder-volumes/volume1```

8. 减少逻辑卷(和扩展一样，根据参数不同有两种方式)
   ```lvreduce -L [大小] [逻辑卷路径] 例如: 1.减少到4G lvreduce -L 4G /dev/cinder-volumes/volume1 ; 2.减少4G lvreduce -L -4G /dev/cinder-volumes/volume1```

9. 删除逻辑卷(要先确保该卷没有被挂载使用)
   ```lvremove [逻辑卷路径] 例如: lvremove /dev/cinder-volumes/volume1```

*Note：有时候删除逻辑卷时会报：Can't remove open logical volume 的错误，这时需要先清除一下linux系统中块设备映射关系,例如：   dmsetup remove -f /dev/mapper/cinder–volumes-volume1*

10. lvm driver在创建卷后还会为卷创建iscsi target,为后续挂卷使用，当cinder的实现用的是TgtAdm(此外还有IteAdm和LioAdm两种实现)其使用的命令为：
    ```tgt-admin –update [iscsi target名称] 例如：tgt-admin –update iqn.2010-10.org.openstack:volume-2748f2f2-4394-4e6e-af8d-8dd34496c024 (cinder默认iscsi名称的构成是 前缀+卷uuid, 其中前缀默认是iqn.2010-10.org.openstack:)```

11. 查询所有的iscsi target的命令：tgt-admin –show

12. 删除iscsi target的命令：tgt-admin –force –delete [iscsi target名称] 
    例如：tgt-admin –force –delete iqn.2010-10.org.openstack:volume-2748f2f2-4394-4e6e-af8d-8dd34496c024
