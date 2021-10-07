---
title: "Configure StoRM Path Authorization Database"
linkTitle: "Use path-authz.db"
weight: 0
noComment: true
description: >
  StoRM Path Authorization Database gives to the system administrator a way to customize users permissions on internal paths of a storage area: different local users/groups could have different rights on the same path.
---

### Introduction

StoRM Path Authorization Database gives to the system administrator a way to customize users permissions on internal paths of a storage area: different local users/groups could have different rights on the same path. For each SRM request, the authorized VO user is mapped on one local user and group. Knowing this, the system administrator can write a list of ACE (Access Control Entry), evaluated by a selected algorithm, in order to permit or deny to a local user/group a list of operations on a particular path.

The choose of the evaluation algorithm and the definition of the list of ACEs can be done by editing the file:

```
/etc/storm/backend-server/path-authz.db.
```

### Evaluation algorithm

The selected evaluation algorithm is indicated by the property **algorithm**. Its default value is:

```bash
algorithm=it.grid.storm.authz.path.model.PathAuthzAlgBestMatch
```

> NOTE: Currently, this is the unique supported algorithm that can be specified.

To determine if a request succeeds, the *PathAuthzAlgBestMatch* evaluation algorithm processes the ACE list in a computed order:

- only the ACE which have a local UNIX group that matches the requester subject are considered;
- the order of the ACE is defined on the base of distance from the requested SURL's StFN and the path specified within the ACE;
- each ACE is processed until all of the bits of the requester's access have been checked.

The result will be:

- `NOT_APPLICABLE` if there are no ACE matching with the requester.
- `INDETERMINATE` if there is at least one bit not checked.
- `DENY` if there is at least one bit DENIED for the requestor
- `PERMIT` if all the bits are PERMIT 

### Access Control Entries

The default ACE stored into `path-authz.db` authorization file allows every kind of operation on all the storage areas:

```bash
#--------+-----------------------+---------+
# user   | Path     | Permission | ACE     |
# class  |          | mask       | type    |
#--------+-----------------------+---------+
  @ALL@    /          RLWFDMN      permit
```

Each ACE is composed by the following information:

- USER CLASS contains the name of the UNIX group involved or `@ALL@` to match all the groups;
- PATH is the relative sub-path based on the storage area root;
- the PERMISSION MASK contains from one to all the following letters:
    - `W` :       WRITE_FILE              "Write data on existing files"
    - `R` :       READ_FILE               "Read data"
    - `F` :       MOVE/RENAME             "Move a file"
    - `D` :       DELETE                  "Delete a file or a directory"
    - `L` :       LIST_DIRECTORY          "Listing a directory"
    - `M` :       CREATE_DIRECTORY        "Create a directory"
    - `N` :       CREATE_FILE             "Create a new file"
- ACE TYPE value could be `permit` or `deny` and indicates if the selected operations on that path have to be blocked or allowed.

### Example

To properly configure StoRM's Path Authorization DataBase it's useful to have the right knowledge about the users/groups involved with [LCMAPS mapping]({{< ref "/docs/getting-started/backend/lcmaps-mapping.md" >}} "LCMAPS Mapping").

Here we're going to show how to configure a simple path authorization filter on our test VO's storage area: *test.vo*.
The idea is to create:
- a sub-directory *test.vo/PRD* accessible in read/write mode only by users that belong to the local group *testvoprd*;
- a sub-directory *test.vo/USERS* accessible in read/write mode only by users that belong to the local group *testvo* but readable by *testvoprd* users.

Edit:

    /etc/storm/backend-server/path-authz.db

as follow:

```bash
#----------+--------------+------------+--------+
# user     | Path         | Permission | ACE    |
# class    |              | mask       | type   |
#----------+--------------+------------+--------+
testvoprd    /test.vo/PRD    RLWFDMN     permit
@ALL@        /test.vo/PRD    RLWFDMN     deny
testvo       /test.vo/USERS  RLWFDMN     permit
testvoprd    /test.vo/USERS  RL          permit
@ALL@        /test.vo/USERS  RLWFDMN     deny
@ALL@        /               RLWFDMN      permit
```

This ACE list say that: 

- All the users mapped into *testvoprd* group can do any kind of operation into */test.vo/PRD* directory and read/list files into */test.vo/USERS* (1st and 4th ACEs);
- All the users mapped into *testvo* group can do any kind of operation into */test.vo/USERS* directory (3rd ACE);
- All the user mapped into a group different from *testvoprd* and *testvo* can't access */test.vo/PRD* or */test.vo/USERS* (2nd and 5th ACEs); 
- No ACEs are defined for the other storage areas (6th ACE).

The order of the ACEs is important because the evaluation algorithm stops after the first match with minimal distance from `Path`. For example, if we invert the first two rows of our ACE list, nobody will be authorized to read or write into */test.vo/PRD*. 

To apply the changes to path-authz.db you need to restart *storm-backend-server*:

```bash
systemctl restart storm-backend-server
```

#### Users

In order to do some test on the path-authz.db configuration showed above, two users with different VOMS attributes have been used:

* **/C=IT/O=IGI/CN=test0**
* **/C=IT/O=INFN/OU=Personal Certificate/L=CNAF/CN=Enrico Vianello** with "Role=R1"

Supposing the same LCMAPS configuration of [this]({{< ref "/docs/getting-started/backend/lcmaps-mapping.md#example" >}} "LCMAPS Mapping Example") example, user *Enrico Vianello* will be mapped into *testvoprd* local group (it has "Role=R1" between its FQANs). User *test0* has no "Role=R1" then it will be mapped to *testvo* local group.

#### SRM tests

The following tests have been done using:

- [**gfal tool**](https://wiki.chipp.ch/twiki/bin/view/CmsTier3/HowToAccessSe#gfal_tools) for the SRM requests
- [**globus-url-copy**](http://toolkit.globus.org/toolkit/docs/3.2/gridftp/user/globusurlcopy.html) of Globus Toolkit to direct upload/download files
- **acl** package (it's a StoRM prerequisite) to read the ACL set on files/directories

```bash
## Switch to user "Enrico Vianello" ##
$ voms-proxy-init --voms test.vo:/test.vo/Role=R1 --cert vianello-2014.p12

## Create directory /test.vo/tmp ##
$ gfal-mkdir srm://centos6-devel.cnaf.infn.it:8444/test.vo/tmp

## Get ACLs from /test.vo/tmp ##
$ getfacl /storage/test.vo/tmp
getfacl: Removing leading '/' from absolute path names
# file: storage/test.vo/tmp
# owner: storm
# group: storm
user::rwx
group::---
group:testvoprd:r-x
mask::rwx
other::---
## The testvoprd ACL has been added added ##

## Create new local file: ##
$ echo "Hello world!!" > hello.txt

## Upload local file to /test.vo/tmp ##
$ cat hello.txt | gfal-save srm://centos6-devel.cnaf.infn.it:8444/test.vo/tmp/hello.txt

## Check if the uploaded file exists: ##
$ gfal-ls srm://centos6-devel.cnaf.infn.it:8444/test.vo/tmp
hello.txt

## Get ACLs from /test.vo/tmp/hello.txt
$ getfacl /storage/test.vo/tmp/hello.txt 
getfacl: Removing leading '/' from absolute path names
# file: storage/test.vo/tmp/hello.txt
# owner: storm
# group: storm
user::rw-
group::---
group:testvoprd:rw-
mask::rwx
other::---
## The testvoprd ACL has been added added

## Switch to test0 user: ##
$ voms-proxy-init --voms test.vo --cert test0.p12

## Get file content /test.vo/tmp/hello.txt ##
$ gfal-copy srm://centos6-devel.cnaf.infn.it:8444/test.vo/tmp/hello.txt file:///tmp/hello.txt
Copying 1   [DONE]  after 1s 

## Check the updated ACL on /test.vo/tmp/hello.txt ##
$getfacl /storage/test.vo/tmp/hello.txt 
getfacl: Removing leading '/' from absolute path names
# file: storage/test.vo/tmp/hello.txt
# owner: storm
# group: storm
user::rw-
group::---
group:testvo:r--
group:testvoprd:rw-
mask::rwx
other::---
## testvo read permission has been added

## Switch to user "Enrico Vianello" ##
$ voms-proxy-init --voms test.vo:/test.vo/Role=R1 --cert vianello-2014.p12

## Create directory /test.vo/PRD (accessible only to testvoprd users):
$ gfal-mkdir srm://centos6-devel.cnaf.infn.it:8444/test.vo/PRD

## getting ACLs on /test.vo/PRD
$ getfacl /storage/test.vo/PRD/
getfacl: Removing leading '/' from absolute path names
# file: storage/test.vo/PRD/
# owner: storm
# group: storm
user::rwx
group::---
group:testvoprd:r-x
mask::rwx
other::---

## Upload local file to /test.vo/PRD/hello.txt
$ cat hello.txt | gfal-save srm://centos6-devel.cnaf.infn.it:8444/test.vo/PRD/hello.txt

## Check if the uploaded file exists:
$ gfal-ls srm://centos6-devel.cnaf.infn.it:8444/test.vo/PRD
hello.txt

## Get ACLs from /test.vo/PRD/hello.txt
$ getfacl /storage/test.vo/PRD/hello.txt 
getfacl: Removing leading '/' from absolute path names
# file: storage/test.vo/PRD/hello.txt
# owner: storm
# group: storm
user::rw-
group::---
group:testvoprd:rw-
mask::rwx
other::---
## The testvoprd ACL has been added added

## Get uploaded file content:
$ gfal-cat srm://centos6-devel.cnaf.infn.it:8444/test.vo/PRD/hello.txt
Hello World!!

## Switch to test0 user:
$ voms-proxy-init --voms test.vo --cert test0.p12

## Get file content expecting unauthorized failure (1):
$ gfal-cat srm://centos6-devel.cnaf.infn.it:8444/test.vo/PRD/hello.txt
gfal-cat: error: Permission denied

## Get file content expecting unauthorized failure (1):
$ gfal-copy srm://centos6-devel.cnaf.infn.it:8444/test.vo/PRD/hello.txt file:///tmp/hello.txt
Copying 1   [FAILED]  after 2s                                                                                                                                                    gfal-copy: error: Permission denied
```

At this point a question could be raised: how does it works with direct calls from GridFTP? The GridFTP TURLs should be used only within the context of a SRM request (between a srmPtG and a srmRf or between a srmPtP and a srmPd). In this case there's no problem because the SRM request will set the necessary ACL (indipendently from the enforcing approach used). If the GridFTP call is made out of the context of a SRM request the scenary depends on the operations done before and on who has done them, or depends on the Default ACL. It's not sure that the necessary ACL is set (if the enforcing approach is JiT the ACL could even not exist), then the request can fail.

In case of AoT enforcing approach, if almost one user mapped on the same group has already done a SRM request on the resource, then a direct access to the same resource, providing the necessary credentials, will be successful.

For example, at the point of the example above, if user *test0* does a globus-url-copy like the gfal-copy just done, the request will be successful:

```bash
## Switch to user test0 ##

## Check failure on file download passing through StoRM SRM endpoint: ##
$ gfal-copy srm://centos6-devel.cnaf.infn.it:8444/test.vo/PRD/hello.txt file:///tmp/hello.txt
Copying 1   [FAILED]  after 2s                                                                                                                                                    gfal-copy: error: Permission denied
## Resource is not reachable via SRM!

## Check failure on file download by directly accessing via GridFTP ##
$ globus-url-copy -vb gsiftp://centos6-devel.cnaf.infn.it:2811/storage/test.vo/PRD/Hello.txt hello.tx
Source: gsiftp://centos6-devel.cnaf.infn.it:2811/storage/test.vo/PRD/
Dest:   file:///home/vianello/
  Hello.txt  ->  hello.tx

error: globus_ftp_client: the server responded with an error
500 Command failed. : open error: Permission denied
```

## Avoid conflicts with *Default ACL List*

StoRM allows site administrators to set a **Default ACL List** for each storage area (see [backend configuration section](http://italiangrid.github.io/storm/documentation/sysadmin-guide/1.11.8/#beconf) of StoRM's System Administrator guide).

The *Default ACL List* contains one or more pairs of a local user (uid) or group id (gid) and a permission (R, W, RW). All these ACL are automatically added by StoRM to the requested resource, at each read or write request.

Obviously, **if the Default ACL involves users or groups that are used also into the path-authz.db, this configuration can generate conflicts** and unexpected behaviors.

For example, if we add to our *test.vo* storage area the following default ACL:

    STORM_TESTVO_DEFAULT_ACL_LIST=testvo:RW

we will give to all the users with group *testvo* read and write permissions on all the files and directories within the storage area. Default ACL list is propagated onto sub-directories and files when they are created or read by a SRM request. Then, if the site administrator applies it on an already existent filesystem tree, he must manually run a command which fix all the storage area's ACL.

Going back to our example, added the `STORM_TESTVO_DEFAULT_ACL_LIST` and after a re-run of yaim confguration, we can retry to create/read/delete from test.vo, with the following results:

```bash
## Create test0 VOMS proxy for test.vo ##
$ voms-proxy-init --voms test.vo --cert test0.p12

## Check root directory: it's empty ##
$ gfal-ls srm://centos6-devel.cnaf.infn.it:8444/test.vo/

## Attempt to create PRD directory: it should fail! ##
$ gfal-mkdir srm://centos6-devel.cnaf.infn.it:8444/test.vo/PRD
gfal-mkdir: error: Permission denied

## Upload file into test.vo:## 
$ cat hello.txt | gfal-save srm://centos6-devel.cnaf.infn.it:8444/test.vo/Hello.txt

## Switch to user Enrico Vianello ##
$ voms-proxy-init --voms test.vo:/test.vo/Role=R1 --cert vianello-2014.p12
...

## Attempt to create PRD directory: a success is expected!
$ gfal-mkdir srm://centos6-devel.cnaf.infn.it:8444/test.vo/PRD

## Upload file into PRD directory ##
$ cat hello.txt | gfal-save srm://centos6-devel.cnaf.infn.it:8444/test.vo/PRD/Hello.txt

## Switch to test0 user ##
$ voms-proxy-init --voms test.vo --cert test0.p12 

## Download unauthorized file: expected error, success retrieved!!
$ globus-url-copy -vb gsiftp://centos6-devel.cnaf.infn.it:2811/storage/test.vo/PRD/Hello.txt /tmp/Hello.txt
Source: gsiftp://centos6-devel.cnaf.infn.it:2811/storage/test.vo/PRD/
Dest:   file:///tmp/
  Hello.txt

## just to see the different ACL:
$ getfacl /storage/test.vo/PRD/Hello.txt
getfacl: Removing leading '/' from absolute path names
# file: storage/test.vo/PRD/Hello.txt
# owner: storm
# group: storm
user::rw-
group::---
group:testvo:rw-
group:testvoprd:rw-
mask::rwx
other::---
```

As expected, the new ACE:

    group:testvo:rw-

is propagated into all the *test.vo* sub-directories and allow read operations on files that would be unreadable before:

```bash
$ globus-url-copy -vb gsiftp://centos6-devel.cnaf.infn.it:2811/storage/test.vo/PRD/Hello.txt /tmp/Hello.txt
Source: gsiftp://centos6-devel.cnaf.infn.it:2811/storage/test.vo/PRD/
Dest:   file:///tmp/
  Hello.txt
```

So, be careful and check both *Defualt ACL List* and *storm-authz.db* in order to avoid conflicts.



