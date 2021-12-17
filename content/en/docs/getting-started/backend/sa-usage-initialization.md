---
title: "Initialize Storage Area used space from file"
linkTitle: "SA used space initialization"
weight: 0
noComment: true
description: >
  The used-space.ini file allows the system administrator to initialize the status of the Storage Area used space. Instructions on how to initialize a SA used space are given in this section.
---

### Introduction

StoRM maintains the information about the status of managed Storage Areas (such as free, used, busy, available, guaranteed and reserved space) and store them into the DB. Whenever some storage space is consumed or released by creating or deleting files, the status is updated. The storage space status stored into the DB is authorative. The information about the storage space stored into the DB are used also as information source for the Information Provider through the DIP (Dynamic Info Provider). There are cases in which the status of a SA must be initialized, for example when a fresh StoRM installation has to manage a storage space already populated with files, where the space used is not zero.  
There are different methods to initialize the status of a SA, some executed within StoRM (GPFS quota and/or background-DU). In this section it is described how an administrator can initialize the status of a SA by using a configuration file, named `used-space.ini`.

### Configure the `used-space.ini` file

StoRM Backend will load `/etc/storm/backend-server/used-space.ini` file at bootstrap and initialize the used space of newly created SA to its values. Thus, restart the service in order to pick up your changes.

The content of the configuration file is quite simple: a list of sections corresponding to the SA in which are defined the used size and eventually the checktime. For each SA that needs to be initializated there is a section named with the same alias *space-token-description* defined in the `/etc/storm/backend-server/namespace.xml` file. Within the section one can define the following properties:

| Property | Description | Format | Required? |
|:---------|:------------|:-------|:----------|
| usedsize | Used space in the SA expressed in Bytes | Value without digits after the decimal mark | Y |
| checktime | Timestamp of the time wich the used size computation refers to | Date in RFC-2822 format | N |


Here is an exsample of the `used-space.ini` configuration file:

```bat
	[IGI_TOKEN]
	checktime = Thu, 16 Dec 2021 18:02:04 +0100
	usedsize = 32724
	[TAPE_TOKEN]
	checktime = Thu, 16 Dec 2021 18:03:30 +0100
	usedsize = 41054
	[TESTVO_TOKEN]
	usedsize = 17686135
````

This file can be produced either manually or with Puppet.

#### Manual configuration

* write your own `/etc/storm/backend-server/used-space.ini` file adding a section for each storage area you want to initialize;
* as section name use the *space-token-description* value as in the `/etc/storm/backend-server/namespace.xml` file.  
  One can get the same alias even from the `storm-get-space-aliases.sh [-u <db-username>] [-p <db-password>]` command (detailed description reported in the [StoRM utils section][storm-utils]);
* set the value of *usedsize* property as in the example;
* set the value of *checktime* property as in the example. To obtain an RFC-2822 timestamp of the current time you can execute the command `date --rfc-2822`.

[storm-utils]: {{< relref "storm-utils-usage.md#storm-get-space-aliases" >}}

#### Puppet configuration

TBD