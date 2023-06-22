---
title: "StoRM WebDAV"
linkTitle: "WebDAV"
weight: 0
noComment: true
description: >
  StoRM WebDAV service installation, operation and configuration.
---

## Service installation


Grap the latest package from the StoRM repository. See instructions
[here](https://italiangrid.github.io/storm/download.html).

```bash
yum install storm-webdav
```

## Service operation

### Starting and stopping the service

In order to start the service type:

```bash
$ systemct start storm-webdav
```

Stop the service with:

```bash
$ systemctl stop storm-webdav
```

### Service status

To check the service status type:

```bash
$ systemctl status storm-webdav
```

Check that the service responds with:

```bash
$ curl http://localhost:8085/actuator/health -s | jq
{
  "status": "UP"
}
```

Get the service metrics at the following URL:

```bash
$ curl http://localhost:8085/status/metrics?pretty=true -s | jq
{
  "version" : "4.0.0",
  "gauges" : {
    "jvm.gc.G1-Old-Generation.count" : {
      "value" : 0
    },
    "jvm.gc.G1-Old-Generation.time" : {
      "value" : 0
    }
    ...
}
```

### Service logs

The service logs live in the `/var/log/storm/webdav` directory.

Here the following logs are saved:

- `storm-webdav-server.log`, which provides the main service log;
- `storm-webdav-server-access.log`, which provides the http access log.

### Access points

A storage area named `sa` is accessible by default at the URL
`https://hostname:8443/sa`.

If the service is configured such to allow anonymous access, the default URL
is `http://hostname:8085/sa`.

