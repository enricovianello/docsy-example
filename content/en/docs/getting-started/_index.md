---
title: "Getting Started"
linkTitle: "Getting Started"
weight: 2
noComment: true
description: >
  This section provides all the information about deploying and configuring StoRM services.
---

StoRM is a lightweight storage resource manager (SRM) solution developed at [INFN](https://home.infn.it/it/), which powers the Italian Tier-1 data center at [INFN-CNAF](https://www.cnaf.infn.it/), as well as more than 30 other sites.

StoRM has a layered architecture (Figure 1), split between two main components: the StoRM frontend and backend services. The StoRM frontend service implements the SRM interface exposed to client applications and frameworks. The StoRM backend service implements the actual storage management logic by interacting directly with the underlying file system. Data transfer is provided by GridFTP, HTTP/WebDAV and XRootD services accessing directly the file system underlying the StoRM deployment. StoRM WebDAV, besides HTTP data transfer functionality, also provides a WebDAV-based data management interface.

![StoRM typical deployment architecture](/storm-docs/images/storm-architecture.png "Figure 1. StoRM typical deployment architecture")

## Prerequisites

### Platform

All the StoRM components are certified to work on [CentOS 7](https://www.centos.org).
There are no specific minimum hardware requirements but it is advisable to have at least 4GB of RAM on Backend host.

### ACL support

StoRM uses the ACLs on files and directories to implement the security model. To do this, StoRM uses the native access to the file system. Therefore, in order to ensure a proper running, ACLs need to be enabled on the underlying file-system and work properly (sometimes they are enabled by default).

If the _getfacl_ and _setfacl_ commands are not available on your host you have to install _acl_ package:

```shell
yum install acl
```

To check if all works properly, try to set an acl to a test file as follow:

```shell
touch test
setfacl -m u:storm:rw test
```

**Note**: `storm` local user must exist.

```shell
getfacl test
```

Should return the following values:

```
# file: test
# owner: root
# group: root
user::rw-
user:storm:rw-
group::r--
mask::rw-
other::r--
```

To enable ACLs (if needed), you must add the acl property to the relevant file system in your `/etc/fstab` file.
For example:

```
/dev/hda3     /storage      ext3     defaults, acl     1 2
```

Then you need to remount the affected partitions as follows:

```shell
mount -o remount /storage
```

This is valid for different file system types (i.e., ext3, xfs, gpfs and others).

### Extended Attribute support

StoRM uses the Extended Attributes (EA) on files to store some metadata related to the file (e.g. the checksum value); therefore in order to ensure a proper running, the EA support needs to be enabled on the underlying file system and work properly.

If the _getfattr_ and _setfattrl_ commands are not available on your host, install `attr` package:

```
yum install attr
```

To check if all properly works, try to set an extendend attribute to a test file:

```
touch testfile
setfattr -n user.testea -v test testfile
getfattr -d testfile
```

It should return:

```
# file: testfile
user.testea="test"
```

To enable EA (if needed) you must add the `user_xattr` property to the relevant file systems in your `/etc/fstab` file.
For example:

```
/dev/hda3     /storage     ext3     defaults,acl,user_xattr     1 2
```

Then you need to remount the affected partitions as follows:

```
mount -o remount /storage
```

### FQDN Hostname

Hostname must be a *Fully Qualified Domain Name* (FQDN).

To check if your hostname is a FQDN, run:

```
hostname -f
```

The command must return the host FQDN.

If you need to correct it and you are using bind or NIS for host lookups, you can change the FQDN and the DNS domain name, which is part of the FQDN, in the /etc/hosts file.

```
# Do not remove the following line, or various programs
# that require network functionality will fail.
127.0.0.1       MYHOSTNAME.MYDOMAIN MYHOSTNAME localhost.localdomain localhost
::1             localhost6.localdomain6 localhost6
```

Set your own MYHOSTNAME and MYDOMAIN and restart the network service:

```
service network restart
```

### Host credentials

Hosts participating to the StoRM-SE which run services such as StoRM Frontend, StoRM Backend, StoRM WebDAV or StoRM Globus GridFTP must be configured with X.509 certificates signed by a trusted Certification Authority (CA).

Usually, the **hostcert.pem** and **hostkey.pem** certificate and private key are located in the `/etc/grid-security` directory. They must have permission 0644 and 0400 respectively:

```bash
ls -l /etc/grid-security/hostkey.pem
-r-------- 1 root root 887 Mar  1 17:08 /etc/grid-security/hostkey.pem

ls -l /etc/grid-security/hostcert.pem
-rw-r--r-- 1 root root 1440 Mar  1 17:08 /etc/grid-security/hostcert.pem
```

Check if your certificate is expired as follow:

```bash
openssl x509 -checkend 0 -in /etc/grid-security/hostcert.pem
```

To change permissions, if necessary:

```bash
chmod 0400 /etc/grid-security/hostkey.pem
chmod 0644 /etc/grid-security/hostcert.pem
```

### NTP service

NTP service must be installed and running.

```
yum install ntp
systemctl enable ntpd
systemctl start ntpd
```