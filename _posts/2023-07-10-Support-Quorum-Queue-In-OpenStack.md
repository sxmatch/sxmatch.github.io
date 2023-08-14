---
layout: post
category : OpenStack
tagline: "keep simple"
tags : [OpenStack, Rabbitmq]
---
{% include JB/setup %}

**If need to reprint, please indicate the source**

**Copyright: 王昊 Hao Wang @sxmatch**

*2023/5/29*

-------
---

# Support Quorum Queue In OpenStack

- Important differences from Classic mirrored queue
  
  1. Regular queues can be non-durable. Quorum queues are always durable.
  
  2. Exclusive queues are tied to the lifecycle of their declaring connection.Quorum queues by design are replicated and durable, therefore quorum queues cannot be exclusive.
  
  3. Certain failure scenarios can result in mirrored queues confirming messages too early, potentially resulting in data loss. Quorum queues are designed to be safer and provide simpler, well-defined failure handling semantics.

- Use Cases of Quorum Queues:
  
  1. Their intended use is for topologies where queues exist for a long time and are critical to certain aspects of system operation, 
     therefore fault tolerance and data safety are more important than the lowest possible latency and advanced queue features.
  
  2. Publishers should use publisher confirms as this is how clients can interact with the quorum queue consensus system.
     Publisher confirms will only be issued once a published message has been successfully replicated to a quorum of nodes and is considered "safe" within the context of the system.
  
  3. Consumers should use manual acknowledgements to ensure messages that aren't successfully processed are returned to
     the queue so that another consumer can re-attempt processing.
  
  4. When a quorum queue is declared, an initial number of replicas for it must be started in the cluster. By default the number of replicas to be started is up to three, one per RabbitMQ node in the cluster.

- When not to use quorum queues:
  
  1. Temporary nature of queues: transient or exclusive queues, high queue churn (declaration and deletion rates).
  
  2. Lowest possible latency: the underlying consensus algorithm has an inherently higher latency due to its data safety features.
  
  3. When data safety is not a priority.
  
  4. Very long queue backlogs.

- How to config OpenStack to support quorum queue
  
  There will be four configurations added to [oslo_messaging_rabbit] group in OpenStack project's conf file.
  
  1. rabbit_quorum_queue=True, open the quorum queue;
  
  2. rabbit_quorum_delivery_limit=0,   limit of redelivery message count before dropped;
  
  3. rabbit_quroum_max_memory_length=0, limit the number of messages in the quorum queue to reduce the memory pressure.
  
  4. rabbit_quroum_max_memory_length=0, limit the number of memory bytes in the quorum queue to reduce the memory pressure.
