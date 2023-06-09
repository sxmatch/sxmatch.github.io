---
layout: post
category : OpenStack
tagline: "keep simple"
tags : [OpenStack, operation]
---
{% include JB/setup %}

**If need to reprint, please indicate the source**

**Copyright: 王昊 Hao Wang @sxmatch**

*2013/07/11*

-------
---

这篇笔记的背景是在开发基于OpenStack的云操作系统产品时，经常会在项目中遇到时间跳变的问题，现网交付或者运维人员常常无法准确评估时间跳变对于系统的影响，因此将时间跳变对于系统的影响分为不同的节点角色类型，总结如下：

1. 控制节点：
    - MQ集群发生脑裂
       影响范围：OpenStack系统自身 不影响已有虚拟机
       影响后果：云操作系统无法正常下发业务
    - keepalived集群
      影响范围：OpenStack系统自身 不影响已有虚拟机
      影响后果：浮动IP无法切换或者生效 云操作系统对外服务失效
    - 数据库集群
      影响范围：OpenStack系统自身 不影响已有虚拟机
      影响后果：数据库集群异常 数据无法入库
    - OpenStack组件
      影响范围：OpenStack系统自身 不影响已有虚拟机
      影响后果：nova cinder neutron等组件内的定时任务异常，组件功能发生异常
    - Portal服务
      影响范围：云操作系统自身 不影响已有虚拟机
      影响后果：portal系统自身定时任务异常，无法正常提供门户服务

2. 计算节点
    - OpenStack组件
      影响范围：计算节点自身 不影响已有虚拟机
      影响后果：nova-compute neutron-agent内部定时任务异常导致资源刷新和状态刷新上报异常，service服务状态为down。
    - 存储集群
      影响范围：存储集群自身Monitor和OSD组件 影响已有虚拟机
      影响后果：存储集群健康状态异常，无法正常读写数据
    - Linux系统Crontab任务
      影响范围：FitOS所有节点 不影响已有虚拟机
      影响后果：Crontab配置的定时任务异常无法正常执行

3.  《时间跳变引起云操作系统异常恢复指导》大纲：
  1. 前置检查
  1.1 检查所有节点上ntp进程是否正常
  1.2 检查所有节点上ntp.conf中时钟源Ip是否一致
  1.3 检查时钟源Ip网络连接正常 时钟源服务正常
  2. 控制节点恢复指导
  2.1 MQ集群恢复指导
  2.2 keepalived集群恢复指导
  2.3 数据库集群恢复指导
  2.4 OpenStack组件恢复指导
  2.5 Portal组件恢复指导
  3. 计算节点恢复指导
  3.1 OpenStack组件恢复指导
  4. 存储集群
  4.1存储集群恢复指导
  5. Linux系统
  5.1 Crontab定时任务恢复指导
