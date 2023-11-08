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

1. Create 3 nodes of cluster VM with Centos-stream-9 image, and configure the /etc/hosts to short name and make sure every node could be connected by hostname.

2. Because several features (e.g. [quorum queues](https://www.rabbitmq.com/quorum-queues.html), [client tracking in MQTT](https://www.rabbitmq.com/mqtt.html)) require a consensus between cluster members, odd numbers of cluster nodes are highly recommended: 1, 3, 5, 7 and so on. Two node clusters are **highly recommended against** since it's impossible for cluster nodes to identify a majority and form a consensus in case of connectivity loss. For example, when the two nodes lose connectivity MQTT client connections won't be accepted, quorum queues would lose their availability, and so on.

3. Install RabbitMQ and Cloudsmith Signing Keys
   
   ```shell
   rpm --import 'https://github.com/rabbitmq/signing-keys/releases/download/3.0/rabbitmq-release-signing-key.asc'
   rpm --import 'https://github.com/rabbitmq/signing-keys/releases/download/3.0/cloudsmith.rabbitmq-erlang.E495BB49CC4BBE5B.key'
   rpm --import 'https://github.com/rabbitmq/signing-keys/releases/download/3.0/cloudsmith.rabbitmq-server.9F4587F226208342.key'
   ```

4. Add Yum Repositories for RabbitMQ and Modern Erlang
   
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

5. Install Rabbitmq and dependencies
   
   ```shell
   dnf update -y
   dnf install socat logrotate -y
   dnf install -y erlang rabbitmq-server
   ```

6. Run Rabbitmq server on every node of cluster
   
   ```shell
   systemctl enable rabbitmq-server
   systemctl start rabbitmq-server
   ```

7. Copy Erlang cookies from node1 to other nodes.
   
   ```shell
   scp .erlang.cookie rabbit-mq-cluster-2:/var/lib/rabbitmq/
   scp .erlang.cookie rabbit-mq-cluster-3:/var/lib/rabbitmq/
   ```

8. Create the cluster, join rabbit node 2 and rabbit node 3 to cluster.
   
   ```shell
   # on rabbit2
   rabbitmqctl stop_app
   # => Stopping node rabbit@rabbit2 ...done.
   
   rabbitmqctl reset
   # => Resetting node rabbit@rabbit2 ...
   
   rabbitmqctl join_cluster rabbit@rabbit-mq-cluster-1
   # => Clustering node rabbit@rabbit2 with [rabbit@rabbit1] ...done.
   
   rabbitmqctl start_app
   # => Starting node rabbit@rabbit2 ...done.
   # Do those steps again on rabbit node 3.
   ```

9. Check the status of cluster
   
   ```shell
   # on rabbit1
   rabbitmqctl cluster_status
   
   # Cluster status of node rabbit@rabbit-mq-cluster-1 ...
   # Basics
   
   # Cluster name: rabbit@rabbit-mq-cluster-1
   # Total CPU cores available cluster-wide: 12
   
   # Disk Nodes
   
   # rabbit@rabbit-mq-cluster-1
   # rabbit@rabbit-mq-cluster-2
   # rabbit@rabbit-mq-cluster-3
   
   # Running Nodes
   
   # rabbit@rabbit-mq-cluster-1
   # rabbit@rabbit-mq-cluster-2
   # rabbit@rabbit-mq-cluster-3
   
   # Versions
   
   # rabbit@rabbit-mq-cluster-1: RabbitMQ 3.12.8 on Erlang 26.1.2
   # rabbit@rabbit-mq-cluster-2: RabbitMQ 3.12.8 on Erlang 26.1.2
   # rabbit@rabbit-mq-cluster-3: RabbitMQ 3.12.8 on Erlang 26.1.2
   
   # CPU Cores
   
   # Node: rabbit@rabbit-mq-cluster-1, available CPU cores: 4
   # Node: rabbit@rabbit-mq-cluster-2, available CPU cores: 4
   # Node: rabbit@rabbit-mq-cluster-3, available CPU cores: 4
   
   # Maintenance status
   
   # Node: rabbit@rabbit-mq-cluster-1, status: not under maintenance
   # Node: rabbit@rabbit-mq-cluster-2, status: not under maintenance
   # Node: rabbit@rabbit-mq-cluster-3, status: not under maintenance
   
   # Alarms
   
   # (none)
   
   # Network Partitions
   
   # (none)
   
   # Listeners
   
   # Node: rabbit@rabbit-mq-cluster-1, interface: [::], port: 25672, protocol: clustering, purpose: inter-node and CLI tool communication
   # Node: rabbit@rabbit-mq-cluster-1, interface: [::], port: 5672, protocol: amqp, purpose: AMQP 0-9-1 and AMQP 1.0
   # Node: rabbit@rabbit-mq-cluster-2, interface: [::], port: 25672, protocol: clustering, purpose: inter-node and CLI tool communication
   # Node: rabbit@rabbit-mq-cluster-2, interface: [::], port: 5672, protocol: amqp, purpose: AMQP 0-9-1 and AMQP 1.0
   # Node: rabbit@rabbit-mq-cluster-3, interface: [::], port: 25672, protocol: clustering, purpose: inter-node and CLI tool communication
   # Node: rabbit@rabbit-mq-cluster-3, interface: [::], port: 5672, protocol: amqp, purpose: AMQP 0-9-1 and AMQP 1.0
   
   # Feature flags
   
   # Flag: classic_mirrored_queue_version, state: enabled
   # Flag: classic_queue_type_delivery_support, state: enabled
   # Flag: direct_exchange_routing_v2, state: enabled
   # Flag: feature_flags_v2, state: enabled
   # Flag: implicit_default_bindings, state: enabled
   # Flag: listener_records_in_ets, state: enabled
   # Flag: maintenance_mode_status, state: enabled
   # Flag: quorum_queue, state: enabled
   # Flag: restart_streams, state: enabled
   # Flag: stream_queue, state: enabled
   # Flag: stream_sac_coordinator_unblock_group, state: enabled
   # Flag: stream_single_active_consumer, state: enabled
   # Flag: tracking_records_in_ets, state: enabled
   # Flag: user_limits, state: enabled
   # Flag: virtual_host_metadata, state: enabled
   
   # on rabbit2
   rabbitmqctl cluster_status
   # Cluster status of node rabbit@rabbit-mq-cluster-2 ...
   # Basics
   
   # Cluster name: rabbit@rabbit-mq-cluster-2
   # Total CPU cores available cluster-wide: 12
   
   # Disk Nodes
   
   # rabbit@rabbit-mq-cluster-1
   # rabbit@rabbit-mq-cluster-2
   # rabbit@rabbit-mq-cluster-3
   
   # Running Nodes
   
   # rabbit@rabbit-mq-cluster-1
   # rabbit@rabbit-mq-cluster-2
   # rabbit@rabbit-mq-cluster-3
   
   # Versions
   
   # rabbit@rabbit-mq-cluster-2: RabbitMQ 3.12.8 on Erlang 26.1.2
   # rabbit@rabbit-mq-cluster-1: RabbitMQ 3.12.8 on Erlang 26.1.2
   # rabbit@rabbit-mq-cluster-3: RabbitMQ 3.12.8 on Erlang 26.1.2
   
   # CPU Cores
   
   # Node: rabbit@rabbit-mq-cluster-2, available CPU cores: 4
   # Node: rabbit@rabbit-mq-cluster-1, available CPU cores: 4
   # Node: rabbit@rabbit-mq-cluster-3, available CPU cores: 4
   
   # Maintenance status
   
   # Node: rabbit@rabbit-mq-cluster-2, status: not under maintenance
   # Node: rabbit@rabbit-mq-cluster-1, status: not under maintenance
   # Node: rabbit@rabbit-mq-cluster-3, status: not under maintenance
   
   # Alarms
   
   # (none)
   
   # Network Partitions
   
   # (none)
   
   # Listeners
   
   # Node: rabbit@rabbit-mq-cluster-2, interface: [::], port: 25672, protocol: clustering, purpose: inter-node and CLI tool communication
   # Node: rabbit@rabbit-mq-cluster-2, interface: [::], port: 5672, protocol: amqp, purpose: AMQP 0-9-1 and AMQP 1.0
   # Node: rabbit@rabbit-mq-cluster-1, interface: [::], port: 25672, protocol: clustering, purpose: inter-node and CLI tool communication
   # Node: rabbit@rabbit-mq-cluster-1, interface: [::], port: 5672, protocol: amqp, purpose: AMQP 0-9-1 and AMQP 1.0
   # Node: rabbit@rabbit-mq-cluster-3, interface: [::], port: 25672, protocol: clustering, purpose: inter-node and CLI tool communication
   # Node: rabbit@rabbit-mq-cluster-3, interface: [::], port: 5672, protocol: amqp, purpose: AMQP 0-9-1 and AMQP 1.0
   
   # Feature flags
   
   # Flag: classic_mirrored_queue_version, state: enabled
   # Flag: classic_queue_type_delivery_support, state: enabled
   # Flag: direct_exchange_routing_v2, state: enabled
   # Flag: feature_flags_v2, state: enabled
   # Flag: implicit_default_bindings, state: enabled
   # Flag: listener_records_in_ets, state: enabled
   # Flag: maintenance_mode_status, state: enabled
   # Flag: quorum_queue, state: enabled
   # Flag: restart_streams, state: enabled
   # Flag: stream_queue, state: enabled
   # Flag: stream_sac_coordinator_unblock_group, state: enabled
   # Flag: stream_single_active_consumer, state: enabled
   # Flag: tracking_records_in_ets, state: enabled
   # Flag: user_limits, state: enabled
   # Flag: virtual_host_metadata, state: enabled
   
   # on rabbit3
   rabbitmqctl cluster_status
   # Cluster status of node rabbit@rabbit-mq-cluster-3 ...
   # Basics
   
   # Cluster name: rabbit@rabbit-mq-cluster-3
   # Total CPU cores available cluster-wide: 12
   
   # Disk Nodes
   
   # rabbit@rabbit-mq-cluster-1
   # rabbit@rabbit-mq-cluster-2
   # rabbit@rabbit-mq-cluster-3
   
   # Running Nodes
   
   # rabbit@rabbit-mq-cluster-1
   # rabbit@rabbit-mq-cluster-2
   # rabbit@rabbit-mq-cluster-3
   
   # Versions
   
   # rabbit@rabbit-mq-cluster-3: RabbitMQ 3.12.8 on Erlang 26.1.2
   # rabbit@rabbit-mq-cluster-1: RabbitMQ 3.12.8 on Erlang 26.1.2
   # rabbit@rabbit-mq-cluster-2: RabbitMQ 3.12.8 on Erlang 26.1.2
   
   # CPU Cores
   
   # Node: rabbit@rabbit-mq-cluster-3, available CPU cores: 4
   # Node: rabbit@rabbit-mq-cluster-1, available CPU cores: 4
   # Node: rabbit@rabbit-mq-cluster-2, available CPU cores: 4
   
   # Maintenance status
   
   # Node: rabbit@rabbit-mq-cluster-3, status: not under maintenance
   # Node: rabbit@rabbit-mq-cluster-1, status: not under maintenance
   # Node: rabbit@rabbit-mq-cluster-2, status: not under maintenance
   
   # Alarms
   
   # (none)
   
   # Network Partitions
   
   # (none)
   
   # Listeners
   
   # Node: rabbit@rabbit-mq-cluster-3, interface: [::], port: 25672, protocol: clustering, purpose: inter-node and CLI tool communication
   # Node: rabbit@rabbit-mq-cluster-3, interface: [::], port: 5672, protocol: amqp, purpose: AMQP 0-9-1 and AMQP 1.0
   # Node: rabbit@rabbit-mq-cluster-1, interface: [::], port: 25672, protocol: clustering, purpose: inter-node and CLI tool communication
   # Node: rabbit@rabbit-mq-cluster-1, interface: [::], port: 5672, protocol: amqp, purpose: AMQP 0-9-1 and AMQP 1.0
   # Node: rabbit@rabbit-mq-cluster-2, interface: [::], port: 25672, protocol: clustering, purpose: inter-node and CLI tool communication
   # Node: rabbit@rabbit-mq-cluster-2, interface: [::], port: 5672, protocol: amqp, purpose: AMQP 0-9-1 and AMQP 1.0
   
   # Feature flags
   
   # Flag: classic_mirrored_queue_version, state: enabled
   # Flag: classic_queue_type_delivery_support, state: enabled
   # Flag: direct_exchange_routing_v2, state: enabled
   # Flag: feature_flags_v2, state: enabled
   # Flag: implicit_default_bindings, state: enabled
   # Flag: listener_records_in_ets, state: enabled
   # Flag: maintenance_mode_status, state: enabled
   # Flag: quorum_queue, state: enabled
   # Flag: restart_streams, state: enabled
   # Flag: stream_queue, state: enabled
   # Flag: stream_sac_coordinator_unblock_group, state: enabled
   # Flag: stream_single_active_consumer, state: enabled
   # Flag: tracking_records_in_ets, state: enabled
   # Flag: user_limits, state: enabled
   # Flag: virtual_host_metadata, state: enabled
   ```

## Replication Mechanism of Rabbitmq

1. All data/state required for the operation of a RabbitMQ broker is replicated across all nodes. An exception to this are message queues, which by default reside on one node, though they are visible and reachable from all nodes. To replicate queues across nodes in a cluster, use a queue type that supports replication. This topic is covered in the [Quorum Queues](https://www.rabbitmq.com/quorum-queues.html) guide.

2. Every queue in RabbitMQ has a primary replica. That replica is called *queue leader* (originally "queue master"). All queue operations go through the leader replica first and then are replicated to followers (mirrors). This is necessary to guarantee FIFO ordering of messages.

3. Assuming all cluster members are available, a messaging (AMQP 0-9-1, AMQP 1.0, MQTT, STOMP) client can connect to any node and perform any operation. Nodes will route operations to the [quorum queue leader](https://www.rabbitmq.com/quorum-queues.html) or [queue leader replica](https://www.rabbitmq.com/ha.html#leader-migration-data-locality) transparently to clients.
   
   With all supported messaging protocols a client is only connected to one node at a time.
   
   In case of a node failure, clients should be able to reconnect to a different node, recover their topology and continue operation. For this reason, most client libraries accept a list of endpoints (hostnames or IP addresses) as a connection option. The list of hosts will be used during initial connection as well as connection recovery, if the client supports it.

4. RabbitMQ brokers tolerate the failure of individual nodes. Nodes can be started and stopped at will, as long as they can contact a cluster member node known at the time of shutdown.
   
   [Quorum queue](https://www.rabbitmq.com/quorum-queues.html) allows queue contents to be replicated across multiple cluster nodes with parallel replication and a predictable [leader election](https://www.rabbitmq.com/clustering.html#quorum-queues.html#leader-election) and [data safety](https://www.rabbitmq.com/quorum-queues.html#data-safety) behavior as long as a majority of replicas are online.
   
   Non-replicated classic queues can also be used in clusters. Non-mirrored queue [behaviour in case of node failure](https://www.rabbitmq.com/ha.html#non-mirrored-queue-behavior-on-node-failure) depends on [queue durability](https://www.rabbitmq.com/queues.html#durability).
   
   RabbitMQ clustering has several modes of dealing with [network partitions](https://www.rabbitmq.com/partitions.html), primarily consistency oriented. Clustering is meant to be used across LAN. It is not recommended to run clusters that span WAN.
