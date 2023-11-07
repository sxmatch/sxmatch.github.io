---
layout: post
category: Database
tagline: "keep simple"
tags: [Database, OpenStack]
---
{% include JB/setup %}

**If need to reprint, please indicate the source**

**Copyright: 王昊 Hao Wang @sxmatch**

*2023/10/30*

-------
---

# Install PostgresSQL Cluster with Docker

1. Create VMs for DB cluster, and install Docker if not existing
   
   ```shell
   sudo yum install -y yum-utils
   sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
   sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
   ```

2. Install Postgres with docker command:
   
   ```shell
   docker pull postgres
   ```

3. Create DB server node 1, 2 and 3
   
   ```shell
   docker run --name pg_node_1 -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=postgres -e POSTGRES_DB=postgres -p 6551:5432 -d postgres
   docker run --name pg_node_2 -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=postgres -e POSTGRES_DB=postgres -p 6551:5432 -d postgres
   docker run --name pg_node_3 -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=postgres -e POSTGRES_DB=postgres -p 6551:5432 -d postgres
   ```

4. 
