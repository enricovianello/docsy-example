---
title: "Configure storm.properties"
linkTitle: "storm.properties"
weight: 1
noComment: true
description: >
  Learn what are the properties that administrators can tune for StoRM Backend.
---

The file `/etc/storm/backend-server/storm.properties` contains StoRM Backend configuration.
This file is read at service startup. Your changes will be applied after a service restart.

> Since StoRM v1.12.0, a huge name refactoring has been done. All the names of the properties, for example, have been cleared of dot and minus.
> The unique separator character allowed is the underscore. The section related to the database connection, in particular, has been also heavily refactored.

| Property | Description | Default |
|:-----|:------------|:--------|
| <td colspan=2>**SRM Service configuration** </td>
| `srm_public_host` | StoRM SRM default host used to initialize SURLs starting from SFN and also to publish info. | It's initialized with the local FQDN |
| `srm_public_port` | StoRM SRM default port used to initialize SURLs starting from SFN and also to publish info. | 8444 |
| `srm_endpoints` | List of comma separated strings identifying the accepted StoRM SRM end-points. Each end-point must be composed of [host]:[port]. This is used by StoRM to understand if a SURL is valid (managed or not). | It's initialized with `srm_public_host`:`srm_public_port` |
| <td colspan=2> **Database connection configuration** </td>
| `db_username` | The connection user name to be passed to the JDBC driver to establish a connection. | "storm" |
| `db_password` | The connection password to be passed to the JDBC driver to establish a connection. | "storm" |
| `db_hostname` | Fully Qualified Domain Name of database hostname. | It's initialized with the local FQDN |
| `db_port` | Database connection URL port. | 3306 |
| `db_properties` | The connection properties that will be sent to the JDBC driver when establishing new connections. Format of the string must be \[propertyName=property;\]\* NOTE - The "user" and "password" properties will be passed explicitly, so they do not need to be included here. | "serverTimezone=UTC&autoReconnect=true" |
| <td colspan=2> **Database connection pool configuration** </td>
| `db_pool_size` | The maximum number of active connections that can be allocated from database connection pool at the same time, or negative for no limit. | -1 |
| `db_pool_min_idle` | The minimum number of connections that can remain idle in the pool, without extra ones being created, or zero to create none. | 10 |
| `db_pool_max_wait_millis` | The maximum number of milliseconds that the pool will wait (when there are no available connections) for a connection to be returned before throwing an exception, or -1 to wait indefinitely.| 5000 |
| `db_pool_test_on_borrow` | The indication of whether objects will be validated before being borrowed from the pool. If the object fails to validate, it will be dropped from the pool, and we will attempt to borrow another. | true |
| `db_pool_test_while_idle` | The indication of whether objects will be validated by the idle object evictor (if any). If an object fails to validate, it will be dropped from the pool. | true |
| <td colspan=2> **REST Services configuration** </td>
| `rest_services_port` | Internal REST services endpoint port. | 9998 |
| `rest_services_max_threads` | Internal REST services endpoint max active requests. | 100 |
| `rest_services_max_queue_size` | Internal REST services endpoint max queue size of accepted requests. | 1000 |
| <td colspan=2> **Sanity Check configuration** </td>
| `sanity_check_enabled` | Enable/disable sanity checks during bootstrap phase. | true |
| <td colspan=2> **XMLRPC Server configuration** </td>
| `xmlrpc_server_port` | Port to listen on for incoming XML-RPC connections from Frontends(s). | 8080 |
| `xmlrpc_max_threads` | Number of threads managing XML-RPC connection from Frontends(s). A well sized value for this parameter have to be at least equal to the sum of the number of working threads in all FrontEend(s). | 256 |
| `xmlrpc_max_queue_size` | Max number of accepted and queued XML-RPC connection from Frontends(s). | 1000 |
| `xmlrpc_security_enabled` | Whether the backend will require a token to be present for accepting XML-RPC requests. | false |
| `xmlrpc_security_token` | The token that the backend will require to be present for accepting XML-RPC requests. | |
| <td colspan=2> **Disk Usage Service Configuration** </td>
| `du_service_enabled` | Disk Usage service is used for the periodic update of the used-space of all the storage spaces that are not GPFS-with-quota-enabled. Running a periodic 'du -s -b' on the top of the storage spaces root directory, the used-space stored into the database is updated. By default, the service is disabled. Disk Usage service is used for the periodic update of the used-space of all the storage spaces that are not GPFS-with-quota-enabled. Running a periodic 'du -s -b' on the top of the storage spaces root directory, the used-space stored into the database is updated. By default, the service is disabled. | false
| `du_service_parallel_tasks` | Enable parallel execution of the expected du. | false |
| `du_service_initial_delay` | The initial delay before the service is started (seconds). | 60 |
| `du_service_tasks_interval` | The interval in seconds between successive run. | 86400 |
| <td colspan=2> **Advanced Configuration** </td>
| `directory_automatic_creation` | Enable/disable the automatic directory creation during srmPrepareToPut requests. | false |
| `directory_writeperm` | Enable/disable write permission on directory created through srmMkDir requests. | false |
| `pinlifetime_default` | Default pinLifetime in seconds used for pinning files in case of srmPrepareToPut or srmPrapareToGet requests | 259200 |
| `pinlifetime_maximum` | Maximum allowed value for pinLifeTime. Values beyond the max will be dropped to max value. | 1814400 |
| `extraslashes_file` | Add extra slashes after the “authority” part of a TURL for FILE protocol. | '' |
| `extraslashes_rfio` | Add extra slashes after the “authority” part of a TURL for RFIO protocol. | '' |
| `extraslashes_gsiftp` | Add extra slashes after the “authority” part of a TURL for GSIFTP protocol. | '/' |
| `extraslashes_root` | Add extra slashes after the “authority” part of a TURL for ROOT protocol. | '/' |
| `gc_pinnedfiles_cleaning_delay` | Initial delay in seconds before starting the garbage collector thread | 10 |
| `gc_pinnedfiles_cleaning_interval` | Garbage Collector time interval in seconds. | 300 |
| `filesize_default` | Default FileSize. | 1000000 |
| `filelifetime_default` | Default FileLifetime in seconds used for VOLATILE file in case of SRM request. | 259200 |
| `default_overwrite` | Default file overwrite mode to use upon srmPrepareToPut requests. Possible values are N (Never), A (Always), D (when files Differs). | A |
| `default_storagetype` | Default File Storage Type to be used for srmPrepareToPut requests. Possible values are  V (Volatile), P (Permanent) and  D (Durable) | P |   
| `scheduler_crusher_worker_core_pool_size` | Crusher Scheduler worker pool base size. | 10 |
| `scheduler_crusher_worker_max_pool_size` | Crusher Schedule worker pool max size. | 50 |
| `scheduler_crusher_queue_size` | Request queue maximum size. | 2000 |
| `scheduler_chunksched_ptp_worker_core_pool_size` | PrepareToPut worker pool base size. | 50 |
| `scheduler_chunksched_ptp_worker_max_pool_size` | PrepareToPut worker pool max size. | 200 |
| `scheduler_chunksched_ptp_queue_size` | PrepareToPut request queue maximum size. | 1000 |
| `scheduler_chunksched_ptg_worker_core_pool_size` | PrepareToGet worker pool base size. | 50 |
| `scheduler_chunksched_ptg_worker_max_pool_size` | PrepareToGet worker pool max size. | 200 |
| `scheduler_chunksched_ptg_queue_size` | PrepareToGet request queue maximum size. | 2000 |
| `scheduler_chunksched_bol_worker_core_pool_size` | BringOnline worker pool base size. | 50 |
| `scheduler_chunksched_bol_worker_max_pool_size` | BringOnline Worker pool max size. | 200 |
| `scheduler_chunksched_bol_queue_size` | BringOnline request queue maximum size. | 2000 |
| `asynch_picking_initial_delay` | Initial delay before starting to pick requests from the DB, in seconds. | 1 |
| `asynch_picking_time_interval` | Polling interval in seconds to pick up new SRM requests. | 2 |
| `asynch_picking_max_batch_size` | Maximum number of requests picked up at each polling time. | 100 |
| `synch_ls_max_entries` | Maximum number of entries returned by an srmLs call. Since in case of recursive srmLs results can be in order of million, this prevent a server overload. | 2000 |
| `synch_ls_default_all_level_recursive` | Default value for the parameter "allLevelRecursive" of the srmLS request. | false |
| `synch_ls_default_num_levels` | Default value for the parameter "numOfLevels" of the srmLS request. | 1 |
| `synch_ls_default_offset` | Default value for the parameter "offset" of the LS request. | 0 |
| `purging` | Enable/Disable Garbage Collector | true |
| `purge_interval` | Time interval for between two requests in garbage collection. In seconds. | 600 |
| `purge_delay` | Initial delay before starting the requests garbage collection process, in seconds. | 10 |
| `purge_size` | Number of requests removed at each run. Every run purge max 800 requests in final status | 800 |
| `expired_request_time` | Time after that the GC consider a _terminated_ request as garbage in seconds | 21600 |
| `expired_inprogress_time` | Time in seconds to consider an in-progress ptp request as expired. | 2592000 |
| `transit_interval` | Expired-Put-Requests-Agent transits expired put requests to a final state. A put request is expired if pinLifetime is reached. This interval is the interval between two agent executions. In seconds. | 300 |
| `transit_delay` | Delay on starting agent. In seconds. | 10 |
| `ptg_skip_acl_setup` | Skip ACL setup for PtG requests | false |
| `health_bookkeeping_enabled` | | false |
| `health_performance_measuring_enabled` | | false |
| `health_electrocardiogram_period` | | 60 |
| `health_performance_logbook_time_interval` | | 15 |
| `health_performance_glance_time_interval` | | 15 |
| `info_quota_refresh_period` | GPFS Quota Manager refresh period (in seconds). | 900 |
| `http_turl_prefix` | URL path prefix for the generated HTTP TURLs. Example: '/subpath'. | '' |
| `server_pool_status_check_timeout` | Timeout for status requests on GridFTP and HTTP endpoints used to generate TURLs. | 20000 |
| `abort_maxloop` | | 10 |
| `ping_properties_filename` | The name of a properties file with extra key=value properties that will be returned during Ping requests. | "ping-values.properties" |

