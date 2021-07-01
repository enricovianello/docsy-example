---
title: "StoRM Security"
linkTitle: "StoRM security"
date: 2021-06-25
weight: 1
noComment: true
description: >
---

### StoRM Authentication And Authorization

StoRM Backend and StoRM WebDAV components are able to support VOMS extension, and to use that to define access policy (complete VOMS-awareness).
StoRM WebDAV service also supports OAuth tokens authentication and authorization.

TO-DO

### The SRM authentication and authorization flow

TO-DO

There are several steps StoRM does to manage access to file:

1. User makes a request with his proxy (hopefully with VOMS extensions)
2. StoRM checks if the user can perform the requested operation on the required resource
3. StoRM ask user mapping to the LCMAPS service
4. StoRM enforce a real ACL on the file and directories requested
5. Jobs running on behalf of the user can perform a direct access on the data

#### StoRM default ACL

StoRM also allows to define default ACLs, a list of ACL entries that will be applied automatically on each read (srmPrepareToGet) and write (srmPrepareToPut) operation. This is useful in case of experiment ise cases, such as the CMS one, that want to allow local access to file on group different from the one that made the SRM request operation. These default ACLs have to be set up on the desired storage area in the namespace.xml file.

TO-DO

### The OAuth Token authentication and authorization flow

TO-DO