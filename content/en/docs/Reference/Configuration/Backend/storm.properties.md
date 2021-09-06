---
title: "Configure StoRM Backend service"
linkTitle: "Service configuration"
weight: 1
noComment: true
description: >
  The storm.properties file contains the StoRM Backend service configuration. In this section you will find all the properties that administrators can tune.
---

The file `/etc/storm/backend-server/storm.properties` contains StoRM Backend configuration.
This file is read at service startup. Your changes will be applied after a service restart.

> Since StoRM v1.12.0, a huge name refactoring has been done. All the names of the properties, for example, have been cleared of dot and minus.
> The unique separator character allowed is the underscore. The section related to the database connection, in particular, has been also heavily refactored.

| Property | Description | Default |
|:-----|:------------|:--------|
| **SRM Service**
| `srm_endpoints` | List of the accepted StoRM SRM end-points. Each end-point must be composed by `host` and `port`. This is used by StoRM to understand if a SURL is valid (managed or not). The first endpoint of this list is considered the default public SRM endpoint. Example: <br/> `srm_endpoints.1.host = fe-storm.example.org` | It's initialized with the local FQDN as host and 8444. |
| ~~`storm.service.FE-public.hostname`~~ | \[*DEPRECATED*\] StoRM Frontend hostname in case of a single Frontend StoRM deployment, StoRM Frontends DNS alias in case of a multiple Frontends StoRM deployment. | |
| ~~`storm.service.port`~~ | \[*DEPRECATED*\] SRM service port. | 8444
| ~~`storm.service.SURL.endpoint`~~ | \[*DEPRECATED*\] List of comma separated strings identifying the StoRM Frontend endpoint(s). This is used by StoRM to understand if a SURL is local. E.g. *srm://storm.cnaf.infn.it:8444/srm/managerv2*. <br/> If you want to accept SURL with the ip address instead of the FQDN hostname you have to add the proper endpoint (E.g. IPv4: *srm://192.168.100.12:8444/srm/managerv2* or IPv6: *srm://[2001:0db8::1428:57ab]:8444/srm/managerv2*. | "srm://\[storm.service.FE-public.hostname\]:8444/srm/managerv2" |
| ~~`storm.service.SURL.default-ports`~~ | \[*DEPRECATED*\] List of comma separated valid SURL port numbers. | 8444
| **Database connection**
| `db.username` | The connection user name to be passed to the JDBC driver to establish a connection. | "storm" |
| `db.password` | The connection password to be passed to the JDBC driver to establish a connection. | "storm" |
| `db.hostname` | Fully Qualified Domain Name of database hostname. | It's initialized with the local FQDN |
| `db.port` | Database connection URL port. | 3306 |
| `db.properties` | The connection properties that will be sent to the JDBC driver when establishing new connections. Format of the string must be \[propertyName=property;\]\* NOTE - The "user" and "password" properties will be passed explicitly, so they do not need to be included here. | "serverTimezone=UTC&autoReconnect=true" |
| **Database connection pool**
| `db.pool.size` | The maximum number of active connections that can be allocated from database connection pool at the same time, or negative for no limit. | -1 |
| `db.pool.min_idle` | The minimum number of connections that can remain idle in the pool, without extra ones being created, or zero to create none. | 10 |
| `db.pool.max_wait_millis` | The maximum number of milliseconds that the pool will wait (when there are no available connections) for a connection to be returned before throwing an exception, or -1 to wait indefinitely.| 5000 |
| `db.pool.test_on_borrow` | The indication of whether objects will be validated before being borrowed from the pool. If the object fails to validate, it will be dropped from the pool, and we will attempt to borrow another. | true |
| `db.pool.test_while_idle` | The indication of whether objects will be validated by the idle object evictor (if any). If an object fails to validate, it will be dropped from the pool. | true |
| **REST Service**
| `rest.port` | Internal REST services endpoint port. | 9998 |
| `rest.max_threads` | Internal REST services endpoint max active requests. | 100 |
| `rest.max_queue_size` | Internal REST services endpoint max queue size of accepted requests. | 1000 |
| **Sanity Checks**
| `sanity_checks_enabled` | Enable/disable sanity checks during bootstrap phase. | true |
| **XMLRPC Server**
| `xmlrpc.port` | Port to listen on for incoming XML-RPC connections from Frontends(s). | 8080 |
| `xmlrpc.max_threads` | Number of threads managing XML-RPC connection from Frontends(s). A well sized value for this parameter have to be at least equal to the sum of the number of working threads in all FrontEend(s). | 256 |
| `xmlrpc.max_queue_size` | Max number of accepted and queued XML-RPC connection from Frontends(s). | 1000 |
| `security_enabled` | Whether the backend will require a token to be present for accepting XML-RPC requests. | false |
| `security_token` | The token that the backend will require to be present for accepting XML-RPC requests. | |
| **Disk Usage Service**
| `du.enabled` | Disk Usage service is used for the periodic update of the used-space of all the storage spaces that are not GPFS-with-quota-enabled. Running a periodic 'du -s -b' on the top of the storage spaces root directory, the used-space stored into the database is updated. By default, the service is disabled. Disk Usage service is used for the periodic update of the used-space of all the storage spaces that are not GPFS-with-quota-enabled. Running a periodic 'du -s -b' on the top of the storage spaces root directory, the used-space stored into the database is updated. By default, the service is disabled. | false
| `du.enable_parallel_tasks` | Enable parallel execution of the expected du. | false |
| `du.initial_delay` | The initial delay before the service is started (seconds). | 60 |
| `du.tasks_interval` | The interval in seconds between successive run. | 86400 |
| **Advanced Configuration**
| `directories.automatic_creation` | Enable/disable the automatic directory creation during srmPrepareToPut requests. | false |
| `directories.writeperm` | Enable/disable write permission on directory created through srmMkDir requests. | false |
| `pinlifetime.default` | Default pinLifetime in seconds used for pinning files in case of srmPrepareToPut or srmPrapareToGet requests | 259200 |
| `pinlifetime.maximum` | Maximum allowed value for pinLifeTime. Values beyond the max will be dropped to max value. | 1814400 |
| `extraslashes.file` | Add extra slashes after the “authority” part of a TURL for FILE protocol. | '' |
| `extraslashes.rfio` | Add extra slashes after the “authority” part of a TURL for RFIO protocol. | '' |
| `extraslashes.gsiftp` | Add extra slashes after the “authority” part of a TURL for GSIFTP protocol. | '/' |
| `extraslashes.root` | Add extra slashes after the “authority” part of a TURL for ROOT protocol. | '/' |
| `gc_pinnedfiles_cleaning_delay` | Initial delay in seconds before starting the garbage collector thread | 10 |
| `gc_pinnedfiles_cleaning_interval` | Garbage Collector time interval in seconds. | 300 |
| `files.default_filesize` | Default FileSize. | 1000000 |
| `files.default_lifetime` | Default FileLifetime in seconds used for VOLATILE file in case of SRM request. | 259200 |
| `files.default_overwrite` | Default file overwrite mode to use upon srmPrepareToPut requests. Possible values are N (Never), A (Always), D (when files Differs). | A |
| `files.default_storagetype` | Default File Storage Type to be used for srmPrepareToPut requests. Possible values are  V (Volatile), P (Permanent) and  D (Durable) | P |   
| `requests_scheduler.core_pool_size` | Crusher Scheduler worker pool base size. | 10 |
| `requests_scheduler.max_pool_size` | Crusher Schedule worker pool max size. | 50 |
| `requests_scheduler.queue_size` | Request queue maximum size. | 2000 |
| `ptp_scheduler.core_pool_size` | PrepareToPut worker pool base size. | 50 |
| `ptp_scheduler.max_pool_size` | PrepareToPut worker pool max size. | 200 |
| `ptp_scheduler.queue_size` | PrepareToPut request queue maximum size. | 1000 |
| `ptg_scheduler.core_pool_size` | PrepareToGet worker pool base size. | 50 |
| `ptg_scheduler.max_pool_size` | PrepareToGet worker pool max size. | 200 |
| `ptg_scheduler.queue_size` | PrepareToGet request queue maximum size. | 2000 |
| `bol_scheduler.core_pool_size` | BringOnline worker pool base size. | 50 |
| `bol_scheduler.max_pool_size` | BringOnline Worker pool max size. | 200 |
| `bol_scheduler.queue_size` | BringOnline request queue maximum size. | 2000 |
| `requests_picker_agent.delay` | Initial delay before starting to pick requests from the DB, in seconds. | 1 |
| `requests_picker_agent.interval` | Polling interval in seconds to pick up new SRM requests. | 2 |
| `requests_picker_agent.max_fetched_size` | Maximum number of requests picked up at each polling time. | 100 |
| `synch_ls.max_entries` | Maximum number of entries returned by an srmLs call. Since in case of recursive srmLs results can be in order of million, this prevent a server overload. | 2000 |
| `synch_ls.default_all_level_recursive` | Default value for the parameter "allLevelRecursive" of the srmLS request. | false |
| `synch_ls.default_num_levels` | Default value for the parameter "numOfLevels" of the srmLS request. | 1 |
| `synch_ls.default_offset` | Default value for the parameter "offset" of the LS request. | 0 |
| `completed_requests_agent.enabled` | Enable/Disable Garbage Collector | true |
| `completed_requests_agent.delay` | Initial delay before starting the requests garbage collection process, in seconds. | 10 |
| `completed_requests_agent.interval` | Time interval for between two requests in garbage collection. In seconds. | 600 |
| `completed_requests_agent.purge_size` | Number of requests removed at each run. Every run purge max 800 requests in final status | 800 |
| `completed_requests_agent.purge_age` | Time after that the GC consider a _terminated_ request as garbage in seconds | 21600 |
| `inprogress_requests_agent.delay` | Delay on starting agent. In seconds. | 10 |
| `inprogress_requests_agent.interval` | Expired-Put-Requests-Agent transits expired put requests to a final state. A put request is expired if pinLifetime is reached. This interval is the interval between two agent executions. In seconds. | 300 |
| `inprogress_requests_agent.ptp_expiration_time` | Time in seconds to consider an in-progress ptp request as expired. | 2592000 |
| `skip_ptg_acl_setup` | Skip ACL setup for PtG requests | false |
| `hearthbeat.bookkeeping_enabled` | | false |
| `hearthbeat.performance_measuring_enabled` | | false |
| `hearthbeat.period` | | 60 |
| `hearthbeat.performance_logbook_time_interval` | | 15 |
| `hearthbeat.performance_glance_time_interval` | | 15 |
| `info_quota_refresh_period` | GPFS Quota Manager refresh period (in seconds). | 900 |
| `http_turl_prefix` | URL path prefix for the generated HTTP TURLs. Example: '/subpath'. | '' |
| `server_pool_status_check_timeout` | Timeout for status requests on GridFTP and HTTP endpoints used to generate TURLs. | 20000 |
| `abort_maxloop` | | 10 |
| `ping_properties_filename` | The name of a properties file with extra key=value properties that will be returned during Ping requests. | "ping-values.properties" |

