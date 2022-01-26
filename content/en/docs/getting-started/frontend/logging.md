---
title: "Logging StoRM FE service information"
linkTitle: "Logging"
weight: 0
noComment: true
description: >
  This section describes how to handle the log file of the StoRM FE service.
---

The Frontend logs information on the service status and the SRM requests received and managed by the process. The Frontend's log supports different level of logging (ERROR, WARNING, INFO, DEBUG, DEBUG2) that can be set via the `log.debuglevel` property in the _storm-frontend-server.conf_ configuration file (placed in the _/etc/storm/frontend-server_ directory). The default logging level is set to INFO.
The Frontend log file named _storm-frontend-server.log_ is placed in the _/var/log/storm_ directory. At start-up time, the FE prints here the whole set of configuration parameters, this can be useful to check the desired values. When a new SRM request is managed, the FE logs information about the user (DN and FQANs) and the requested parameters.
At each SRM request, the FE logs also this important information:

```bat
03/19 11:51:42 0x88d4ab8 main: AUDIT - Active tasks: 3
03/19 11:51:42 0x88d4ab8 main: AUDIT - Pending tasks: 0
```

about the status of the worker pool threads and the pending process queue. _Active tasks_ is the number of worker threads actually running. _Pending tasks_ is the number of SRM requests queued in the worker pool queue. These data gives important information about the Frontend load.

### The monitoring.log file

The monitoring service can be enable/disable with the `monitoring.enabled` property (default value is true).
It provides information about the operations executed in a certain amount of time, which are stored on a _monitoring.log_ file available in the  _/var/log/storm_ directory. The time period (called Monitoring Round) can be set via the configuration property ```monitoring.timeInterval```; its default value is 1 minute. At each Monitoring Round, a single row is printed on
log. This row reports both information about requests that have been performed in the last Monitoring Round and information considering the whole FE execution time (Aggregate Monitoring). Informations reported are generated from both Synchronous and Asynchronous requests and tell the user:

- how many requests have been performed in the last Monitoring Round,
- how many of them were successful,
- how many failed,
- how many produced an error,
- the average execution time,
- the minimum execution time,
- the maximum execution time.

This row reports the **Monitoring Summary** and this is the default behavior of the monitoring service.

**_Example_**:

```bat
03/20 14:19:11 : [# 22927 lifetime=95:33:18] S [OK:47,F:15,E:0,m:0.085,M:3.623,Avg:0.201] A [OK:16,F:0,E:0,m:0.082,M:0.415,Avg:0.136]
   Last:(S [OK:12,F:5,E:0,m:0.091,M:0.255] A [OK:6,F:0,E:0,m:0.121,M:0.415])
```

Furthermore a more detailed Frontend Monitoring activity can be requested by setting the configuration property ```monitoring.detailed``` to _true_. Doing this, at each Monitoring Round for each kind of SRM operation performed in the Monitoring Round (srmls, srmPtp, srmRm, ...) the following information are printed in a section with header "_Last round details:_":

- how many request succeeded,
- how many failed,
- how many produced an error,
- the average execution time,
- the minimum execution time,
- the maximum execution time,
- the execution time standard deviation.

This is called the **Detailed Monitoring Round**. After this, the Monitoring Summary is printed. Then, considering the whole Frontend execution time, in a section with header "Details:", a similar detailed summary is printed. This is called the **Aggregate Detailed Monitoring**.

**_Example_**:

```bat
03/20 14:19:11 : Last round details:
03/20 14:19:11 : [PTP] [OK:3,F:0,E:0,Avg:0.203,Std Dev:0.026,m:0.183,M:0.240]
03/20 14:19:11 : [Put done] [OK:2,F:0,E:0,Avg:0.155,Std Dev:0.018,m:0.136,M:0.173]
03/20 14:19:11 : [# 22927 lifetime=95:33:18] S [OK:47,F:15,E:0,m:0.085,M:3.623,Avg:0.201] A [OK:16,F:0,E:0,m:0.082,M:0.415,Avg:0.136]
  Last:(S [OK:12,F:5,E:0,m:0.091,M:0.255] A [OK:6,F:0,E:0,m:0.121,M:0.415])
03/20 14:19:11 : Details:
03/20 14:19:11 : [PTP] [OK:7,F:0,E:0,Avg:0.141,Std Dev:0.057,m:0.085,M:0.240]
03/20 14:19:11 : [Put done] [OK:5,F:0,E:0,Avg:0.152,Std Dev:0.027,m:0.110,M:0.185]
03/20 14:19:11 : [Release files] [OK:4,F:0,E:0,Avg:0.154,Std Dev:0.044,m:0.111,M:0.216]
03/20 14:19:11 : [Rm] [OK:3,F:0,E:0,Avg:0.116,Std Dev:0.004,m:0.111,M:0.122]
```

**Note**:

- Operations not performed in current Monitoring Round are not printed in Detailed Monitoring Round.
- Operations never performed are not printed in Aggregate Detailed Monitoring.
- Operation performed in current Monitoring Round are aggregated in Aggregate Detailed Monitoring.

### gSOAP tracefile

If you have problem at gSOAP level, and you have already looked at the troubleshooting section of the StoRM site without finding a solution, and you are brave enough, you could try to find some useful information on the gSOAP log file.  
To enable gSOAP logging, add a drop-in file of the `storm-frontend-server` unit:

- create a _*.config_ file (for example _gsoap.conf_) in the _/etc/systemd/system/storm-frontend-server.service.d_ directory;
- add the following content
  ```bat
  [Service]
  Environment="CGSI_TRACE=1"
  Environment="CGSI_TRACEFILE=/tmp/tracefile"
  ```
  where _/tmp/tracefile_ can be customized with any path to your log file;
- reload all the unit files with `systemctl daemon-reload`;
- restart the server with `systemctl restart storm-frontend-server`.

Please be very careful, it prints really a huge amount of information.
