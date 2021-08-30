---
title: "Pool accounts configuration"
linkTitle: "Pool accounts"
date: 2021-06-25
weight: 5
noComment: true
description: >

---

### Create local pool accounts with YAIM

In order to create your local pool accounts with YAIM, you need to define a `users.conf` file and assign its absolute path to the YAIM variable `USERS_CONF`.

The file *users.conf* contains the list of Linux users (pool accounts) to be created. It's a plain list of the users and their IDs. An example of this configuration file can be found into:

```bash
/opt/glite/yaim/examples/users.conf
```

More details can be found in the [User configuration section in the YAIM guide](https://twiki.cern.ch/twiki/bin/view/LCG/YaimGuide400#User_configuration_in_YAIM).

The UNIX users here defined must be created on the service nodes that need them (mainly CE and WNs). The format is the following (fields must not have any white space):

```bash
UID:LOGIN:GID1[,GID2,...]:GROUP1[,GROUP2,...]:VO:FLAG:
```

- UID = user ID. This must be a valid uid. Make sure the number you choose is not assigned to another user.
- LOGIN = login name
- GID1 = primary group ID. This must be a valid gid. Make sure the number you choose is not assigned to another group.
- GID2 = secondary group ID.
- GROUP1 = primary group
- GROUP2 = secondary group
- VO = virtual organization
- FLAG = string to identify special users, further described below

#### Example

We want to create the following pool accounts:
- a pool account of 100 users with group name *testvo* and *test.vo* as VO name
- a pool account of 30 production users with group name *testvoprd*, *test.vo* as VO name and *prd* as FLAG.

In order to do this we have to create the following file:

* `storm-users.conf`

This file contains the definition of all the users (with their relative groups):

```bash
71001:tstvo001:7100:testvo:test.vo::
71002:tstvo002:7100:testvo:test.vo::
71003:tstvo003:7100:testvo:test.vo::
71004:tstvo004:7100:testvo:test.vo::
...
71100:tstvo100:7100:testvo:test.vo::
71101:testvoprd001:7170,7100:testvoprd,testvo:test.vo:prd:
71102:testvoprd002:7170,7100:testvoprd,testvo:test.vo:prd:
71103:testvoprd003:7170,7100:testvoprd,testvo:test.vo:prd:
71104:testvoprd004:7170,7100:testvoprd,testvo:test.vo:prd:
...
71129:testvoprd029:7170,7100:testvoprd,testvo:test.vo:prd:
71130:testvoprd030:7170,7100:testvoprd,testvo:test.vo:prd:
```

To apply the changes, define YAIM's variables into a site.def configuration file:

```bash
USERS_CONF=/path/to/storm-users.conf
VOS=test.vo
```

You need a proper YAIM profile `/opt/glite/yaim/node-info.d/se_config_users` defined as follows:

```
se_config_users_FUNCTIONS="
config_users
"
```

The only function used in this profile is the one that creates the users.
Then, you can run yaim configuration as follow:

```bash
/opt/glite/yaim/bin/yaim \
    -c -s site.def \
    -n se_config_users
```

> Install YAIM with:
> ```
> yum install -y glite-yaim-core attr
> ```
