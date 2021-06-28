---
title: "Released StoRM v1.11.21"
linkTitle: "StoRM v1.11.21"
date: 2021-05-12
description: >
  Released packages: StoRM Backend 1.11.21, StoRM Frontend 1.8.15, StoRM WebDAV 1.4.1 and StoRM Utils 1.0.0
---

The StoRM Product Team is pleased to announce the release of
[StoRM 1.11.21][release-notes] that includes the following updated components:

* StoRM Backend v. [1.11.21][backend-rn]
* StoRM Frontend v. [1.8.15][frontend-rn]
* StoRM WebDAV v. [1.4.1][webdav-rn]

and introduces:

* StoRM Utils v. [1.0.0][utils-rn]

This release:

* fixes the [known issue][known-issue-post] about the upgrade to StoRM v1.11.20 which could break connections with MariaDB
* fixes the boot order for both Frontend and Backend ensuring that mariadb service is started before StoRM services;
* fixes the failed state shown on stop/restart of Java components;
* provides a set of scripts that can be used to edit from command line the storage space info related to a storage area.

Please, follow the [upgrade instructions][upgrade-instructions].
Read the [release notes][release-notes] for more details.

[backend-rn]: {{site.baseurl}}/release-notes/storm-backend-server/1.11.21/
[frontend-rn]: {{site.baseurl}}/release-notes/storm-frontend-server/1.8.15/
[webdav-rn]: {{site.baseurl}}/release-notes/storm-webdav/1.4.1/
[utils-rn]: {{site.baseurl}}/release-notes/storm-utils/1.0.0/

[release-notes]: {{site.baseurl}}/release-notes/StoRM-v1.11.21.html
[download-page]: {{site.baseurl}}/download.html
[storm-sysadmin-guide]: {{site.baseurl}}/documentation/sysadmin-guide/1.11.21

[upgrade-instructions]: {{site.baseurl}}/documentation/sysadmin-guide/1.11.21/upgrading/

[known-issue-post]: {{site.baseurl}}/2021/04/30/storm-v1.11.20-known-issue.html
