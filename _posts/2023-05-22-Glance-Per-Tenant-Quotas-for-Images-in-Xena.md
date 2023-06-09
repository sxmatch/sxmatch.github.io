---
layout: post
category : OpenStack
tagline: "keep simple"
tags : [OpenStack, Glance]
---
{% include JB/setup %}

**If need to reprint, please indicate the source**

**Copyright: 王昊 Hao Wang @sxmatch**

*2023/5/22*

-------
---

## Glance Per-Tenant Quotas For Image

- Background Introduction
  Glance has supported resource consumption quotas on tenants through the use of Keyston's unified functionality. Resouce limits are registed in Keystone with suitable default values, and the default vlaues could be overridden on a per-tenant basis.
  When a resource consumption attempt is made in Glance, the current consumption is computed and compared against the limit set in Keystone; If the specified limit is been overd, the request will be denied.

- Resource Types of Quota
  *Note: Limits are enforced at the time in which resource consumption is attemped, so setting an existing user's quota for any item below the current usage will prevent them from consuming **more** data until they free up space.*
  
  1. Total Image Size
     The **image_size_total** limit defines the maximum amount of storage (MiB) that the tenant may consume across all of their active images.
  
  2. Total Staging Size
     **image_stage_size** is work with **interoperable image import** function, this limit defines the total amount of staging space that may be used. Glance suggests to provide the user with a very generous image_size_total quota and a relatively restrictive image_stage_total allocation to limit user to import one image at any given point.
  
  3. Total Number of Image
     **image_count_total** limit controls the maximum number of image objects that the user may have, regardless of the individual or collective sizes or impact to storage. This limits is helpful to prevent users from creating thousands of small server snapshots without ever deleting them.
  
  4. Total Number of In-Progress Uploads
     Glance can't enforce the storage-foucused quotas while image data stream is uploading, but it can limit the number of parallel upload operations that can be in-progress at any single point. **image_count_uploading** limit provides the controls to conventional image upload, pre-import stage(web-download/glance-direct methods), and copy-image operations. This limit will protect image storage from malicious user starting multiple simultaneous unbounded upload streams.

- Steps To Configure Glance for Per-Tenant Quotas For Image
  
  1. Prepare a system scoped token
     ```# source /etc/kolla/admin-openrc.sh```
     ```# unset OS_TENANT_NAME OS_PROJECT_NAME OS_PROJECT_DOMAIN_NAME OS_USER_DOMAIN_NAME```
  
  2. Register the quota limits in Keystone:
     ```# openstack --os-system-scope all registered limit create --service glance --default-limit <limit number> --region RegionOne image_size_total```
     ```# openstack --os-system-scope all registered limit create --service glance --default-limit <limit number> --region RegionOne image_stage_total```
     ```# openstack --os-system-scope all registered limit create --service glance --default-limit <limit number> --region RegionOne image_count_total```
     ```# openstack --os-system-scope all registered limit create --service glance --default-limit <limit number> --region RegionOne image_count_uploading```
  
  3. Change Glance's glance-api.conf configuration file to use Keystone quotas
     
     - In [DEFAULT] section, enable per-tenant quotas in/etc/kolla/glance-api/glance-api.conf:
       
       ```
       [DEFAULT]
       use_keystone_limits = True
       ```
     
     - In the [oslo_limit] section, configure access to keystone:
       
       ```
       [oslo_limit]
       auth_url = <keystone_public_endpoint>:5000
       auth_type = password
       user_domain_id = default
       username = glance
       system_scope = all
       password = GLANCE_PASSWORD
       endpoint_id = <Glance_public_endopint_id>
       region_name = RegionOne
       insecure = True
       ```
       
        *NOTE: Glance_PASSWORD need to replace by the password for the **glance** user in the Keystone*.
     
     - Make Sure that glance account has reader access to system-scope resource
       
       ```
       openstack role add --user glance --user-domain Default --system all reader
       ```
     
     - Restart the Glance service.
  
  4. Delete the registered limit
     ```openstack --os-system-scop all registered limit delete <registered limit id>```
  
  5. Change the registered limit
     If user wants to change the registered limit for some resource, this limit should be deleted first, and then create a new one.

- Steps To Configure Image Quota limit for Specific Project
  Glance also support to set image quota limit for specific project after setting the per-tenant quota limit, 
  
  1. Prepare a system scoped token
     ```# source /etc/kolla/admin-openrc.sh```
     ```# unset OS_TENANT_NAME OS_PROJECT_NAME OS_PROJECT_DOMAIN_NAME OS_USER_DOMAIN_NAME```
  
  2. Create project limit for image quota resource
     ```# openstack --os-system-scope all limit create --project <project name> --service glance --resource-limit <limit number> --region RegionOne image_size_total```
     ```# openstack --os-system-scope all limit create --project <project name> --service glance --resource-limit <limit number> --region RegionOne image_stage_total```
     ```# openstack --os-system-scope all limit create --project <project name> --service glance --resource-limit <limit number> --region RegionOne image_count_total```
     ```# openstack --os-system-scope all limit create --project <project name> --service glance --resource-limit <limit number> --region RegionOne image_count_uploading```
  
  3. Update project limit for image quota resource
     ```openstack --os-system-scope all limit set --resource-limit <limit number> <limit-id>```
