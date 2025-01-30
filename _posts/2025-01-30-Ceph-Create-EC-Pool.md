---
layout: post
category : Ceph
tagline: "keep simple"
tags : [Ceph, OpenStack]
---
{% include JB/setup %}

**If need to reprint, please indicate the source**

**Copyright: 王昊 Hao Wang @sxmatch**

*2025/01/30*

-------
---

# Create EC Pool and Integrate with OpenStack

### 1. Create EC Profile

```shell
# ceph osd erasure-code-profile set ssd-ec22 k=2 m=2 cursh-failure-domain=osd crush-device-class=ssd
# ceph osd erasure-code-profile get ssd-ec22
crush-device-class=ssd
crush-failure-domain=host
crush-root=default
cursh-failure-domain=osd
jerasure-per-chunk-alignment=false
k=2
m=2
plugin=jerasure
technique=reed_sol_van
w=8
```

### 2. Create EC Pool

```shell
# ceph osd pool create backup-ec22 erasure ssd-ec22
# ceph osd pool set backup-ec22 allow_ec_overwrites true
# ceph osd pool get backup-ec22 all
size: 4
min_size: 3
pg_num: 1
pgp_num: 1
crush_rule: backup-ec22
hashpspool: true
allow_ec_overwrites: true
nodelete: false
nopgchange: false
nosizechange: false
write_fadvise_dontneed: false
noscrub: false
nodeep-scrub: false
use_gmt_hitset: 1
erasure_code_profile: ssd-ec22
fast_read: 0
pg_autoscale_mode: on
eio: false
bulk: false
```

### 3. Create EC User

```shell
# ceph auth list

# ceph auth get-or-create client.cinder-rbd-ssd-ec mon "profile rbd" \
# mgr "profile rbd pool=volume-ssd-ec22, profile rbd pool=volume-ssd, profile rbd pool=backup, profile rbd pool=backup-ec22, profile rbd pool=image, profile rbd pool=image-ec22" \
# osd "profile rbd pool=volume-ssd-ec22, profile rbd pool=volume-ssd, profile rbd pool=backup, profile rbd pool=backup-ec22, profile rbd pool=image, profile rbd pool=image-ec22"

# ceph auth get-or-create client.glance-rbd-ssd-ec mon "profile rbd" \
# mgr "profile rbd pool=image, profile rbd pool=image-ec22" \
# osd "profile rbd pool=image, profile rbd pool=image-ec22"

# ceph auth caps client.cinder-rbd-ssd-ec \
# mon "profile rbd" mgr "profile rbd pool=volume-ssd-ec22, profile rbd pool=volume-ssd, profile rbd pool=backup, profile rbd pool=backup-ec22, profile rbd pool=image, profile rbd pool=image-ec22" \
# osd "profile rbd pool=volume-ssd-ec22, profile rbd pool=volume-ssd, profile rbd pool=backup, profile rbd pool=backup-ec22, profile rbd pool=image, profile rbd pool=image-ec22"
```

### 4. Change The Crush Rule

```shell
# cephadm shell
# ceph osd getcrushmap -o crushmap.bin            # dump CRUSH rule
# crushtool -d crushmap.bin -o crushmap_new       # decompile rule
# vi crushmap_new                                 # edit
# crushtool -c crushmap_new -o crushmap_new.bin   # compile rule
# ceph osd setcrushmap -i crushmap_new.bin        # load rule
# exit

# ceph osd crush rule dump volume-ssd-ec22
```

### 5. Init the EC Pool

```shell
# rbd pool init volume-ssd-ec22
```

### 6. Integrate With Glance

```shell
# vi glance-api/ceph.conf
[client.glance-rbd-ssd-ec]
rbd_default_data_pool = image-ec22
# vi glance-api/ceph.client.glance-rbd-ssd-ec.keyring
[client.glance-rbd-ssd-ec]
key = AQC41JtnCLPyOBAAMYrFEyekM5pO0SYT4UDQ3Q==
# vi glance-api/config.json
{
  "source": "/var/lib/kolla/config_files/ceph.client.glance-rbd-ssd-ec.keyring",
  "dest": "/etc/ceph/ceph.client.glance-rbd-ssd-ec.keyring",
  "owner": "glance",
  "perm": "0600"
},
# vi glance-api/glance.conf
[glance_store]
default_backend = rbd-ec22

[rbd]
rbd_store_user = infra
rbd_store_pool = image

[rbd-ec22]
rbd_store_user = glance-rbd-ssd-ec
rbd_store_pool = image
```

### 6. Integrate With Nova

```shell
# For nova libvirt:
# uuidgen
# create a secret definition file, like <uuid>.xml
# virsh secret-define --file <uuid>.xml
<secret ephemeral='no' private='no'>
  <uuid>44da95e2-a1ca-4e68-a9de-08b4a5f2fff8</uuid>
  <usage type='ceph'>
    <name>client.cinder-ec secret</name>
  </usage>
</secret>
# virsh secret-set-value --secret <uuid> --base64 <CephX-Key>
# virsh secret-list
# restart libvirt
```

### 7. Integrate With Cinder

```shell
# vi cinder-volume/ceph.conf
[client.cinder-rbd-ssd-ec]
rbd_default_data_pool = volume-ssd-ec22
# vi cinder-volume/ceph.client.cinder-rbd-ssd-ec.keyring
[client.cinder-rbd-ssd-ec]
key = AQDa7ZNnG/cCAhAAbCkgNLoz8/dBohCWlf/Q0w==
# vi cinder-volume/config.json
{
  "source": "/var/lib/kolla/config_files/ceph.client.cinder-rbd-ssd-ec.keyring",
  "dest": "/etc/ceph/ceph.client.cinder-rbd-ssd-ec.keyring",
  "owner": "cinder",
  "perm": "0600",
  "optional": false
},
# vi cinder-api/cinder-scheduler/cinder-volume/cinder.conf
[default]
enabled_backends = rbd-ssd,rbd-hdd,rbd-ssd-ec22

[rbd-ssd-ec22]
volume_driver = cinder.volume.drivers.rbd.RBDDriver
volume_backend_name = rbd-ssd-ec22
rbd_pool = volume-ssd
rbd_ceph_conf = /etc/ceph/ceph.conf
rbd_flatten_volume_from_snapshot = false
rbd_max_clone_depth = 5
rbd_store_chunk_size = 4
rados_connect_timeout = 5
rbd_user = cinder-rbd-ssd-ec
rbd_secret_uuid = 44da95e2-a1ca-4e68-a9de-08b4a5f2fff8
report_discard_supported = True
image_upload_use_cinder_backend = True
```