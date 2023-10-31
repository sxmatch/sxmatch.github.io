---
layout: post
category: Cloud Native
tagline: "keep simple"
tags: [Kubernetes]
---
{% include JB/setup %}

**If need to reprint, please indicate the source**

**Copyright: 王昊 Hao Wang @sxmatch**

*2023/10/30*

-------
---

# Introduce to Kubernetes

## 1. Overview

1. Kubernetes is a portable, extensible, open-source platform for managing containerized workloads and services, that facilitates both declarative configuration and automation. It has a large, rapidly growing ecosystem. Kubernetes services, support, and tools are widely available.

2. Kubernetes is focusing on containerized workloads, which are similar to VM but don't have strict isolation between containers. Therefore, containers are considered lightweight and portable than VMs.

3. Kubernetes provides users with a framework to run distributed systems resiliently. It will take care of scaling and failover for users' applications, extracting some resource concepts to deploy workload more easily.

4. Of course, as a platform for containerized applications, Kubernetes could be deployed to bare metal servers or cloud platforms like OpenStack.

5. A Kubernetes cluster consists of a set of worker nodes, that run containerized applications. Every cluster has at least one work node. The worker nodes host the pods that are the components of the application workload. The control plane manages the worker nodes and the pods in the cluster. For high availability and fault tolerance, the control plane usually runs across multiple computers and a cluster usually runs multiple nodes. There are various components which are needed for a complete and working Kubernetes cluster.
   
   1. kube-apiserver: The API server is a component of the Kubernetes control plane that exposes the Kubernetes API. The API server is the front end for the Kubernetes control plane.
   
   2. etcd: Consistent and highly-available key value store used as Kubernetes' backing store for all cluster data.
   
   3. kube-scheduler: Control plane component that watches for newly created Pods with no assigned node, and selects a node for them to run on.
   
   4. kube-controller-manager: Control plane component that runs controller processes. Including some examples:
      
      - Node Controller: Responsible for noticing and responding when nodes go down.
      
      - Job Controller: Watches for Job objects that represent one-off tasks, then creates Pods to run those tasks to completion.
      
      - EndpointSlice Controller: Populates EndpointSlice objects (to provide a link between Services and Pods).
      
      - ServiceAccount Controller: Create default ServiceAccounts for new namespaces.
   
   5. cloud-controller-manager: A Kubernetes control plane component that embeds cloud-specific control logic, which lets you link your cluster into your cloud provider's API, and separates out the components that interact with that cloud platform from components that only interact with your cluster.
   
   6. kubelet: An agent that runs on each node in the cluster. It makes sure that containers are running in a Pod.
   
   7. kube-proxy: kube-proxy is a network proxy that runs on each node in your cluster, implementing part of the Kubernetes Service concept. It maintains network rules on nodes and allows network communication to your pods from inside or outside of your cluster.
   
   8. container runtime: A fundamental component that empowers Kubernetes to run containers effectively. It is responsible for managing the execution and lifecycle of containers within the Kubernetes environment.

## 2. Workload Resources

### 1. POD

Pods are the smallest deployable and schedulable units of computing that you can create and manage in Kubernetes. A pod is a group of one or more containers, with shared storage and network resources, and a specification for how to run the containers.

### 2. Deployment

A deployment provides declarative updates for Pods and ReplicaSets. The user describes a desired state in Deployment, and the deployment controller changes the actual state to the desired state at a controlled rate. User can define Deployments to create new Replicasets or to remove existing Deployments and adopt all their resources with new Deployments.

### 3. ReplicaSet

A ReplicaSet's purpose is to maintain a stable set of replica Pods running at any given time. As such, it is often used to guarantee the availability of a specified number of identical Pods. However, a Deployment is a higher-level concept that manages ReplicaSets and provides declarative updates to Pods along with a lot of other useful features. Therefore, recommend using Deployments instead of directly using ReplicaSets, unless you require custom update orchestration or don't require updates at all.

### 4. StatefulSet

StatefulSet is the workload API object used to manage stateful applications. Manages the deployment and scaling of a set of Pods, *and provides guarantees about the ordering and uniqueness* of these Pods. Like a Deployment, a StatefulSet manages Pods that are based on an identical container spec. Unlike a Deployment, a StatefulSet maintains a sticky identity for each of its Pods. These pods are created from the same spec, but are not interchangeable: each has a persistent identifier that it maintains across any rescheduling.

### 5. DaemonSet

A *DaemonSet* ensures that all (or some) Nodes run a copy of a Pod. As nodes are added to the cluster, Pods are added to them. As nodes are removed from the cluster, those Pods are garbage collected. Deleting a DaemonSet will clean up the Pods it created.

### 6. Job

A Job creates one or more Pods and will continue to retry execution of the Pods until a specified number of them successfully terminate. As pods successfully complete, the Job tracks the successful completions. When a specified number of successful completions is reached, the task (ie, Job) is complete. Deleting a Job will clean up the Pods it created. Suspending a Job will delete its active Pods until the Job is resumed again.

### 7. CronJob

A *CronJob* creates Jobs on a repeating schedule. CronJob is meant for performing regular scheduled actions such as backups, report generation, and so on. One CronJob object is like one line of a *crontab* (cron table) file on a Unix system. It runs a job periodically on a given schedule, written in Cron format.

### 8. ReplicationController

A *ReplicationController* ensures that a specified number of pod replicas are running at any one time. If there are too many pods, the ReplicationController terminates the extra pods. If there are too few, the ReplicationController starts more pods. Also, users can use Deployment with replica instead of ReplicaitonController. ReplicaSet is the next-generation ReplicationController that supports the new set-based label selector.

## 3. Networking

### 1. Service

In Kubernetes, a Service is a method for exposing a network application that is running as one or more Pods in your cluster. A key aim of Services in Kubernetes is that you don't need to modify your existing application to use an unfamiliar service discovery mechanism. You can run code in Pods, whether this is a code designed for a cloud-native world or an older app you've containerized. You use a Service to make that set of Pods available on the network so that clients can interact with it.

There are three types of Services:

1. ClusterIP: Exposes the Service on a cluster-internal IP. Choosing this value makes the Service only reachable from within the cluster.

2. NodePort: Exposes the Service on each Node's IP at a static port (the `NodePort`). To make the node port available, Kubernetes sets up a cluster IP address, the same as if you had requested a Service of `type: ClusterIP`.

3. LoadBalancer: Exposes the Service externally using an external load balancer. Kubernetes does not directly offer a load-balancing component; you must provide one, or you can integrate your Kubernetes cluster with a cloud provider.

4. ExternalName: Maps the Service to the contents of the `externalName` field (for example, to the hostname `api.foo.bar.example`). The mapping configures your cluster's DNS server to return a `CNAME` record with that external hostname value. No proxying of any kind is set up.

### 2. Ingress

Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. Traffic routing is controlled by rules defined on the Ingress resource. An Ingress may be configured to give Services externally reachable URLs, load balance traffic, terminate SSL / TLS, and offer name-based virtual hosting. An Ingress controller is responsible for fulfilling the Ingress, usually with a load balancer, though it may also configure your edge router or additional frontends to help handle the traffic.

An Ingress does not expose arbitrary ports or protocols. Exposing services other than HTTP and HTTPS to the internet typically uses a service of type Service.Type=NodePort or

Service.Type=LoadBalancer.

### 3. Ingress Controller

In order for the Ingress resource to work, the cluster must have an ingress controller running. Unlike other types of controllers which run as part of the `kube-controller-manager` binary, Ingress controllers are not started automatically with a cluster. Use this page to choose the ingress controller implementation that best fits your cluster.

### 4. EndpointSlices

In Kubernetes, an EndpointSlice contains references to a set of network endpoints. The control plane automatically creates EndpointSlices for any Kubernetes Service that has a selector specified. These EndpointSlices include references to all the Pods that match the Service selector. EndpointSlices group network endpoints together by unique combinations of protocol, port number, and Service name. The name of an EndpointSlice object must be a valid DNS subdomain name. There are a variety of other use cases for EndpointSlices, such as service mesh implementations.

### 5. Network Policy

NetworkPolicies are an application-centric construct which allows you to specify how a pod is allowed to communicate with various network "entities" (we use the word "entity" here to avoid overloading the more common terms such as "endpoints" and "services", which have specific Kubernetes connotations) over the network. NetworkPolicies apply to a connection with a pod on one or both ends and are not relevant to other connections.

The entities that a Pod can communicate with are identified through a combination of the following 3 identifiers:

1. Other pods that are allowed (exception: a pod cannot block access to itself)

2. Namespaces that are allowed

3. IP blocks (exception: traffic to and from the node where a Pod is running is always allowed, regardless of the IP address of the Pod or the node)

### 6. DNS

Kubernetes creates DNS records for Services and Pods. You can contact Services with consistent DNS names instead of IP addresses. Kubernetes publishes information about Pods and Services which is used to program DNS. Kubelet configures Pods' DNS so that running containers can look up Services by name rather than IP. Services defined in the cluster are assigned DNS names. By default, a client Pod's DNS search list includes the Pod's own namespace and the cluster's default domain

## 4. Storage

### 1. Persistent Volume and Persistent Volume Claim

A *PersistentVolume* (PV) is a piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes. It is a resource in the cluster just like a node is a cluster resource. PVs are volume plugins like Volumes but have a lifecycle independent of any individual Pod that uses the PV. This API object captures the details of the implementation of the storage, be that NFS, iSCSI, or a cloud-provider-specific storage system. Kubernetes supports two `volumeModes` of PersistentVolumes: `Filesystem` and `Block`.A *PersistentVolumeClaim* (PVC) is a request for storage by a user. It is similar to a Pod. Pods consume node resources and PVCs consume PV resources. Pods can request specific levels of resources (CPU and Memory). Claims can request specific size and access modes (e.g., they can be mounted ReadWriteOnce, ReadOnlyMany or ReadWriteMany).

### 2. Projected Volume

A `projected` volume maps several existing volume sources into the same directory. Currently, the following types of volume sources can be projected:

1. secret

2. downwardAPI

3. configMap

4. ServiceAccountToken

### 3. Ephemeral Volume

Some applications need additional storage but don't care whether that data is stored persistently across restarts. For example, caching services are often limited by memory size and can move infrequently used data into storage that is slower than memory with little impact on overall performance. Other applications expect some read-only input data to be present in files, like configuration data or secret keys. *Ephemeral volumes* are designed for these use cases. Because volumes follow the Pod's lifetime and get created and deleted along with the Pod, Pods can be stopped and restarted without being limited to where some persistent volume is available.

### 4. Storage Classes

A StorageClass provides a way for administrators to describe the "classes" of storage they offer. Different classes might map to quality-of-service levels, or to backup policies, or to arbitrary policies determined by the cluster administrators. Kubernetes itself is unopinionated about what classes represent. This concept is sometimes called "profiles" in other storage systems.

### 5. Volume Snapshot

A `VolumeSnapshotContent` is a snapshot taken from a volume in the cluster that has been provisioned by an administrator. It is a resource in the cluster just like a PersistentVolume is a cluster resource.

A `VolumeSnapshot` is a request for a snapshot of a volume by a user. It is similar to a PersistentVolumeClaim.

`VolumeSnapshotClass` allows you to specify different attributes belonging to a `VolumeSnapshot`. These attributes may differ among snapshots taken from the same volume on the storage system and therefore cannot be expressed by using the same `StorageClass` of a `PersistentVolumeClaim`.

## 5. Configuration

### 1. ConfigMap

A ConfigMap is an API object used to store non-confidential data in key-value pairs. Pods can consume ConfigMaps as environment variables, command-line arguments, or as configuration files in a volume. A ConfigMap allows you to decouple environment-specific configuration from your container images so that your applications are easily portable.

### 2. Secret

A Secret is an object that contains a small amount of sensitive data such as a password, a token, or a key. Such information might otherwise be put in a Pod specification or in a container image. Using a Secret means that you don't need to include confidential data in your application code. Because Secrets can be created independently of the Pods that use them, there is less risk of the Secret (and its data) being exposed during the workflow of creating, viewing, and editing Pods.

### 6. Security

### 1. Role and ClusterRole

An RBAC(Role-based Access Control) Role or ClusterROle contains rules that represent a set of permissions. A Role always sets permissions within a particular namespace. ClusterRole, by contrast, is a non-namespaced Role. CLusterRoles have several uses:

1. Define permissions on namespaced resource and be granted access within individual namespace(s);

2. Define permissions on namespaced resources and be granted access across all namespaces.

3. Define permissions on cluster-scoped resources.

If the user wants to define a role within a namespace, use a Role; If the user wants to define a role cluster-wide, use a ClusterRole.

### 2. Service Account

A service account is a type of non-human account that, in Kubernetes, provides a distinct identity in a Kubernetes cluster. Application Pods, system components, and entities inside and outside the cluster can use a specific ServiceAccount's credentials to identify as that ServiceAccount. This identity is useful in various situations, including authenticating to the API server or implementing identity-based security policies. When you create a cluster, Kubernetes automatically creates a ServiceAccount object named `default` for every namespace in your cluster.

A service account provides an identity for processes that run in a Pod. A process inside a pod can use the identity of its associated service account to authenticate to the cluster's API server. By default, the Kubernetes control plane adds a projected volume to pods, and this volume includes a token for Kubernetes API access.

### 3. Namespace

In Kubernetes, a Namespace provides a mechanism for isolating groups of API resources within a single cluster. This isolation has two key dimensions:

1. Object names within a namespace can overlap with names in other namespaces, similar to files in folders. This allows tenants to name their resources without having to consider what other tenants are doing.

2. Many Kubernetes security policies are scoped to namespaces. For example, RBAC Roles and Network Policies are namespace-scoped resources. Using RBAC, Users and Service Accounts can be restricted to a namespace.

## 7. Policy

### 1. Resource Quota

A resource quota, defined by a `ResourceQuota` object, provides constraints that limit aggregate resource consumption per namespace. It can limit the number of objects that can be created in a namespace by type, as well as the total amount of compute resources that may be consumed by resources in that namespace.
