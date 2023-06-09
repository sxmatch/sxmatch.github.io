---
layout: post
category : OpenStack
tagline: "keep simple"
tags : [OpenStack, cinder]
---
{% include JB/setup %}

**If need to reprint, please indicate the source**

**Copyright: 王昊 Hao Wang @sxmatch**

*2015/5/30*

-------
---

本文主要介绍Nova和cinder模块中对卷(磁盘)加密特性的实现方案和流程，水平有限，如有错误，请多多指教。

### 什么是加密卷？

是一种卷(磁盘)类型，虚拟机在挂载加密卷后，往卷上写的数据会经过加密算法加密，从而保证卷上数据的安全性。

### 加密卷的特点

加密卷挂载到虚拟机上，当有数据往卷上要写的时候，在virtualization host中对数据进行加密，涉及的模块也有只有
virtualization host 和key management entity。 块存储服务不需要改变。

当然现有工具也支持加密数据(在虚拟机中进行)，但是需要手工配置。Cinder Volume目前提供了很少的安全措施，并且在ISCSI中也无法提供传输数据时的保护措施，所以通过这个特性，数据在传输前已经被加密，并且在存储在磁盘上也是加密的数据。

### 特性流程

整个特性流程分为
