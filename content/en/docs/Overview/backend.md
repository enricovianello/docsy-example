---
title: "StoRM Backend"
linkTitle: "Backend"
noComment: true
description: >
  The Backend is the core of SRM implementation and REST services. It executes all SRM functionalities, it takes care of file and space metadata management, enforces authorization permissions on files and interacts with external Grid services.
---

The StoRM Backend component has several responsabilities:

* executes all synchronous (active) actions
* retrieves asynchronous request from the database
* executes all asynchronous actions
* binds with underlying file systems
* enforces authorization policy on files
* manages SRM file and space metadata

## Internal macro components

StoRM Backend has the following internal macro components:

* Asynchronous SRM requests manager
* Synchronous SRM requests manager
* XML-RPC server
* Persistence manager
* Namespace component
* Autorization component
* Filesystem manager

## SRM requests from database

The Picker component retrieves the specified amount of new SRM requests from the Database at each time interval, and forward them to a Scheduler. The Scheduler takes care of forward the request to the right worker thread as a new task to be executed. The request status is updated into the Database with all the information concerining request results, error and other data. This data are accessible from the FE to answer to a srmStatusOf\* request. This pattern is shown in the following picture:

![Retrieving new SRM requests from database.](/storm-docs/images/reqfromdb.png "Figure 1. Retrieving new SRM requests from database.")

Most of the parameters characterizing this architecture are configurable.

## File System driver

StoRM interacts with the different file systems supported through a driver mechanism. Each driver is a native library written mainly in C/C++, since most of the file system provides C libraries for the advanced API. StoRM Backend uses JNI to connect with drivers. The functionalities provided by each driver are:

* ACL management
* Space management

The drivers available with StoRM are:

* _posixfs_ generic driver for posix file system. It relies on the standard ```setfacl()```, ```getfacl()``` syscall for ACL management, and it does not provide any advanced space management capabilities.
* _GPFS_ specific driver that relies on GPFS advanced API, such as ```gpfs_prealloc``` for space management and ```gpfs_set_acl()``` for ACL management.

This driver mechanism implements a common interface and decouple StoRM internal logic from the different functionalities provided by the underlying storage system. The drivers are loaded at run time following the storage namespace configuration. A single StoRM server is able to work on different file system at the same time, and with this flexible approach it can be easily adapted to support new kind of file systems or other storage resources.