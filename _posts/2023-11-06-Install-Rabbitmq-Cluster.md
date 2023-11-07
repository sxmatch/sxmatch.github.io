---
layout: post
category: Database
tagline: "keep simple"
tags: [Database, OpenStack]
---
{% include JB/setup %}

**If need to reprint, please indicate the source**

**Copyright: 王昊 Hao Wang @sxmatch**

*2023/11/06*

-------
---

# Install Rabbitmq Cluster

## Overview

This document is for indicating how to install and configure the Rbbitmq Cluster by manual.

For production, 3 nodes are required at least.

## Steps of installation

1. Create 3 nodes of cluster VM with Centos-stream-9 image, and configure the /etc/hosts to ensure every node could be connected by hostname.

2. Install RabbitMQ and Cloudsmith Signing Keys
   
   ```shell
   rpm --import 'https://github.com/rabbitmq/signing-keys/releases/download/3.0/rabbitmq-release-signing-key.asc'
   rpm --import 'https://github.com/rabbitmq/signing-keys/releases/download/3.0/cloudsmith.rabbitmq-erlang.E495BB49CC4BBE5B.key'
   rpm --import 'https://github.com/rabbitmq/signing-keys/releases/download/3.0/cloudsmith.rabbitmq-server.9F4587F226208342.key'
   ```

3. Add Yum Repositories for RabbitMQ and Modern Erlang
   
   ```shell
   # Add centos-stream-9 repo file rabbitmq.repo under /etc/yum.repos.d/
   
   # In /etc/yum.repos.d/rabbitmq.repo
   
   ##
   ## Zero dependency Erlang RPM
   ##
   
   [modern-erlang]
   name=modern-erlang-el9
   # uses a Cloudsmith mirror @ yum.novemberain.com.
   # Unlike Cloudsmith, it does not have any traffic quotas
   baseurl=https://yum1.novemberain.com/erlang/el/9/$basearch
           https://yum2.novemberain.com/erlang/el/9/$basearch
           https://dl.cloudsmith.io/public/rabbitmq/rabbitmq-erlang/rpm/el/9/$basearch
   repo_gpgcheck=1
   enabled=1
   gpgkey=https://github.com/rabbitmq/signing-keys/releases/download/3.0/cloudsmith.rabbitmq-erlang.E495BB49CC4BBE5B.key
   gpgcheck=1
   sslverify=1
   sslcacert=/etc/pki/tls/certs/ca-bundle.crt
   metadata_expire=300
   pkg_gpgcheck=1
   autorefresh=1
   type=rpm-md
   
   [modern-erlang-noarch]
   name=modern-erlang-el9-noarch
   # uses a Cloudsmith mirror @ yum.novemberain.com.
   # Unlike Cloudsmith, it does not have any traffic quotas
   baseurl=https://yum1.novemberain.com/erlang/el/9/noarch
           https://yum2.novemberain.com/erlang/el/9/noarch
           https://dl.cloudsmith.io/public/rabbitmq/rabbitmq-erlang/rpm/el/9/noarch
   repo_gpgcheck=1
   enabled=1
   gpgkey=https://github.com/rabbitmq/signing-keys/releases/download/3.0/cloudsmith.rabbitmq-erlang.E495BB49CC4BBE5B.key
          https://github.com/rabbitmq/signing-keys/releases/download/3.0/rabbitmq-release-signing-key.asc
   gpgcheck=1
   sslverify=1
   sslcacert=/etc/pki/tls/certs/ca-bundle.crt
   metadata_expire=300
   pkg_gpgcheck=1
   autorefresh=1
   type=rpm-md
   
   [modern-erlang-source]
   name=modern-erlang-el9-source
   # uses a Cloudsmith mirror @ yum.novemberain.com.
   # Unlike Cloudsmith, it does not have any traffic quotas
   baseurl=https://yum1.novemberain.com/erlang/el/9/SRPMS
           https://yum2.novemberain.com/erlang/el/9/SRPMS
           https://dl.cloudsmith.io/public/rabbitmq/rabbitmq-erlang/rpm/el/9/SRPMS
   repo_gpgcheck=1
   enabled=1
   gpgkey=https://github.com/rabbitmq/signing-keys/releases/download/3.0/cloudsmith.rabbitmq-erlang.E495BB49CC4BBE5B.key
          https://github.com/rabbitmq/signing-keys/releases/download/3.0/rabbitmq-release-signing-key.asc
   gpgcheck=1
   sslverify=1
   sslcacert=/etc/pki/tls/certs/ca-bundle.crt
   metadata_expire=300
   pkg_gpgcheck=1
   autorefresh=1
   
   ## 
   ## RabbitMQ Server
   ##
   
   [rabbitmq-el9]
   name=rabbitmq-el9
   baseurl=https://yum2.novemberain.com/rabbitmq/el/9/$basearch
           https://yum1.novemberain.com/rabbitmq/el/9/$basearch
           https://dl.cloudsmith.io/public/rabbitmq/rabbitmq-server/rpm/el/9/$basearch
   repo_gpgcheck=1
   enabled=1
   
   # Cloudsmith's repository key and RabbitMQ package signing key
   
   gpgkey=https://github.com/rabbitmq/signing-keys/releases/download/3.0/cloudsmith.rabbitmq-server.9F4587F226208342.key
          https://github.com/rabbitmq/signing-keys/releases/download/3.0/rabbitmq-release-signing-key.asc
   gpgcheck=1
   sslverify=1
   sslcacert=/etc/pki/tls/certs/ca-bundle.crt
   metadata_expire=300
   pkg_gpgcheck=1
   autorefresh=1
   type=rpm-md
   
   [rabbitmq-el9-noarch]
   name=rabbitmq-el9-noarch
   baseurl=https://yum2.novemberain.com/rabbitmq/el/9/noarch
           https://yum1.novemberain.com/rabbitmq/el/9/noarch
           https://dl.cloudsmith.io/public/rabbitmq/rabbitmq-server/rpm/el/9/noarch
   repo_gpgcheck=1
   enabled=1
   
   # Cloudsmith's repository key and RabbitMQ package signing key
   
   gpgkey=https://github.com/rabbitmq/signing-keys/releases/download/3.0/cloudsmith.rabbitmq-server.9F4587F226208342.key
          https://github.com/rabbitmq/signing-keys/releases/download/3.0/rabbitmq-release-signing-key.asc
   gpgcheck=1
   sslverify=1
   sslcacert=/etc/pki/tls/certs/ca-bundle.crt
   metadata_expire=300
   pkg_gpgcheck=1
   autorefresh=1
   type=rpm-md
   
   [rabbitmq-el9-source]
   name=rabbitmq-el9-source
   baseurl=https://yum2.novemberain.com/rabbitmq/el/9/SRPMS
           https://yum1.novemberain.com/rabbitmq/el/9/SRPMS
           https://dl.cloudsmith.io/public/rabbitmq/rabbitmq-server/rpm/el/9/SRPMS
   repo_gpgcheck=1
   enabled=1
   gpgkey=https://github.com/rabbitmq/signing-keys/releases/download/3.0/cloudsmith.rabbitmq-server.9F4587F226208342.key
   gpgcheck=0
   sslverify=1
   sslcacert=/etc/pki/tls/certs/ca-bundle.crt
   metadata_expire=300
   pkg_gpgcheck=1
   autorefresh=1
   type=rpm-md
   ```

4. Install Rabbitmq and dependencies
   
   ```shell
   dnf update -y
   dnf install socat logrotate -y
   dnf install -y erlang rabbitmq-server
   ```

5. Run Rabbitmq server on every node of cluster
   
   ```shell
   systemctl enable rabbitmq-server
   ```

6. Start independent nodes
   
   ```shell
   # on rabbit1
   rabbitmq-server -detached
   # on rabbit2
   rabbitmq-server -detached
   # on rabbit3
   rabbitmq-server -detached
   ```

7. Create the cluster, join rabbit node 2 and rabbit node 3 to cluster.
   
   ```shell
   # on rabbit2
   rabbitmqctl stop_app
   # => Stopping node rabbit@rabbit2 ...done.
   
   rabbitmqctl reset
   # => Resetting node rabbit@rabbit2 ...
   
   rabbitmqctl join_cluster rabbit@rabbit1
   # => Clustering node rabbit@rabbit2 with [rabbit@rabbit1] ...done.
   
   rabbitmqctl start_app
   # => Starting node rabbit@rabbit2 ...done.
   # Do those steps again on rabbit node 3.
   ```

8. Check the status of cluster
   
   ```shell
   # on rabbit1
   rabbitmqctl cluster_status
   # => Cluster status of node rabbit@rabbit1 ...
   # => [{nodes,[{disc,[rabbit@rabbit1,rabbit@rabbit2,rabbit@rabbit3]}]},
   # =>  {running_nodes,[rabbit@rabbit3,rabbit@rabbit2,rabbit@rabbit1]}]
   # => ...done.
   
   # on rabbit2
   rabbitmqctl cluster_status
   # => Cluster status of node rabbit@rabbit2 ...
   # => [{nodes,[{disc,[rabbit@rabbit1,rabbit@rabbit2,rabbit@rabbit3]}]},
   # =>  {running_nodes,[rabbit@rabbit3,rabbit@rabbit1,rabbit@rabbit2]}]
   # => ...done.
   
   # on rabbit3
   rabbitmqctl cluster_status
   # => Cluster status of node rabbit@rabbit3 ...
   # => [{nodes,[{disc,[rabbit@rabbit3,rabbit@rabbit2,rabbit@rabbit1]}]},
   # =>  {running_nodes,[rabbit@rabbit2,rabbit@rabbit1,rabbit@rabbit3]}]
   # => ...done.
   ```
