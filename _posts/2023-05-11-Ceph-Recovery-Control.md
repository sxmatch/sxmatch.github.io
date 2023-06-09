---
layout: post
category : Ceph
tagline: "keep simple"
tags : [Ceph, Recovery]
---
{% include JB/setup %}

**If need to reprint, please indicate the source**

**Copyright: 王昊 Hao Wang @sxmatch**

*2023/5/11*

-------
---

# Instruction of Ceph Recovery Limit

Ceph now use two types of queue & queuing sheduler for prioritizing osd operations, wpg and mclock. The WeightedPriorityQueue (`wpq`) dequeues operations in relation to their priorities to prevent starvation of any queue. WPQ should help in cases where a few OSDs are more overloaded than others. The mClockQueue (`mclock_scheduler`) prioritizes operations based on which class they belong to (recovery, scrub, snaptrim, client op, osd subop). In Ceph Pacific release, the default queue is wpg, and after Pacific, it will be mclock. This instruction will introduce two ways to modify ceph recovery limits based on wpg and mclock for keeping operational performances.

- Applicable Scenario
  Ceph recovery will be existing under some scenarios, like OSD daemon crashes and comes back online, replacing OSD's hardware for troubleshooting, or extending/reducing storage capacity of Ceph Cluster, etc. If recovery process is occuring, it could make process time consuming and resource intensive which might block the client IO from user side. The purpose of this instruction is reducing the impact of Ceph recovery process on client IO.

- Introduction of Recovery Configurations
  To maintain operational performance, Ceph can performs the recovery process with limitiations on some configurations.
  There are some config options which would be used:
  
  1. "**osd_max_backfills**":The maximum number of backfills allowed to or from a single OSD when recovery process occur. Default value is 1;
  
  2. "**osd_recovery_sleep**": Time in seconds to sleep before the next recovery or backfill operation. Increasing this value will slow down recovery operation while client operations will be less impacted. Default value is 0.0;
  
  3. "**osd_recovery_max_active**": The maximum number of active recovery requests per OSD at one time. More requests will accelerate recovery, but the requests places an increased load on the cluster. Default value is 0;
  
  4. "**osd_recovery_max_active_hdd**": The number of active recovery requests per OSD at one time, if the primary device is rotational. Default value is 3.
  
  5. "**osd_recovery_max_active_ssd**": The number of active recovery requests per OSD at one time, if the primary device is non-rotational (i.e., an SSD). Default value is 10.
  
  6. "**osd_recovery_max_single_start**": The maximum number of recovery operations per OSD that will be newly started when an OSD is recovering. Default value is 1.
  
  7. "**osd_recovery_op_priority**": Priority of osd recovery operations.Default value is 3;
  
  8. "**osd_recovery_max_chunk**": The maximum total size of data chunks a recovery operation can carry. Default value is 8Mi.
     *NOTE: osd_recovery_max_active's value is only used if it is non-zero. Normally it is `0`, which means that the `hdd` or `ssd` values are used, depending on the type of the primary device backing the OSD.*

- Steps to Modify Recovery Limits of OSDs
  
  1. Ensure and record the default values of those configurations.
     ```# ceph-conf --show-config | egrep "osd_max_backfills|osd_recovery_sleep|osd_recovery_max_active|osd_recovery_op_priority|osd_recovery_max_single_start|osd_recovery_max_chunk"```
  
  2. It will show those information below:
     
     ```
     osd_max_backfills = 1
     osd_recovery_max_active = 0
     osd_recovery_max_active_hdd = 3
     osd_recovery_max_active_ssd = 10
     osd_recovery_max_chunk = 8388608
     osd_recovery_max_single_start = 1
     osd_recovery_op_priority = 3
     osd_recovery_sleep = 0.000000
     osd_recovery_sleep_hdd = 0.100000
     osd_recovery_sleep_hybrid = 0.025000
     osd_recovery_sleep_ssd = 0.000000```
     ```
  
  3. To reduce the impact of recovery process, ceph administrator can adjust those configurations for a better performance. For example:
     ```# ceph tell 'osd.*' injectargs --osd_recovery_max_active_ssd=1 --osd_recovery_max_active_hdd=1 --osd_recovery_sleep=1 --osd_recovery_op_priority=1```
  
  4. In a similar way, it also can be adjusted for accelerating the recovery process if there is need for it. For example:
     ```# ceph tell 'osd.*' injectargs --osd_max_backfills=2 --osd_recovery_max_active=3 --osd_recovery_sleep=0 --osd_recovery_op_priority=63```
     *NOTE: The value of configurations sometimes depends on the situation of ceph cluster,  it might require the administrator to adjust them by experience.*

- Steps to Modify Mclock Max Recovery Limits
  After Pacific release, Ceph use mclock scheduler by default to control client I/O, background recovery, scrub,etc. Administrator could modify the default max backfills or recovery limits if the need arises.
  *NOTE: The recommendation is to retain the defaults as is on a running cluster as modifying them could have unexpected performance outcomes. The values may be modified only if the cluster is unable to cope/showing poor performance with the default settings or for performing experiments on a test cluster.*
  
  1. The modification of the mClock default backfills/recovery limit is gated by the **osd_mclock_override_recovery_settings** option, which is set to _false_ by default. Attempting to modify any default recovery/backfill limits without setting the gating option will reset that option back to the mClock defaults along with a warning message logged in the cluster log.
  
  2. Set the **osd_mclock_override_recovery_settings** option on all osds to true:
     ```# ceph config set osd osd_mclock_override_recovery_settings true```
  
  3. Set the desired max backfill/recovery option:
     ```# ceph config set osd osd_max_backfills <value>```
     ```# ceph config set osd osd_recovery_max_active <value>```
     ```# ceph config set osd osd_recovery_max_active_hdd <value>```
     ```# ceph config set osd osd_recovery_max_active_ssd <value>```
  4. Wait for a few seconds and verify the running configuration for a specific OSD, for example:
     ```# ceph config show osd.N | grep osd_max_backfills```