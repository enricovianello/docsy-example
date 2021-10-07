---
title: "LCMAPS mapping"
linkTitle: "LCMAPS mapping"
date: 2021-06-25
weight: 3
noComment: true
description: >
  StoRM Backend needs a mapping service that converts user's Grid credentials to a local UNIX user and group. The service used by StoRM is **LCMAPS**.
---

LCMAPS is the Local Credential Mapping Service and it takes care of translating grid credentials to Unix credentials local to the site by using [the pool account mechanism](http://www.gridsite.org/gridmapdir/). It takes care of ensuring that different individuals on the Grid remain distinct Unix accounts. Using group mappings based on the user's VO attributes, isolation and scheduling priority decisions can be made. 

> *See more on https://wiki.nikhef.nl/grid/LCMAPS*



This mapping can be configured via YAIM through the variables `USERS_CONF` and `GROUPS_CONF`. These variables contain the full absolute path of a couple of files: *users.conf* and *groups.conf*.

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

You can customize this file to your site needs. 

The *groups.conf* file defines the user categories that must be accepted by the grid services provided by a site. It indicates for each category to which kind of local accounts the user should be mapped, where applicable. The file has the following format:

```bash
"VOMS_FQAN":GROUP:GID:FLAG:[VO]
```

- VOMS_FQAN = VOMS proxy fully qualified attribute name
- GROUP = UNIX group
- GID = UNIX GID
- FLAG = string to identify special users, further described below
- VO = virtual organization (optional. It allows the VO to be specified explicitly, otherwise it will be derived from the VOMS FQAN

The groups.conf distributed by YAIM is only an example. You can remove the lines that doesn't apply to your site or VO and add new lines if needed. Example:

```bash
"/dteam/ROLE=lcgadmin":::sgm:
"/dteam/ROLE=production":::prd:
"/dteam"::::
```

The *groups.conf* file lists the VOMS proxy primary FQANs that are accepted. If a proxy has a secondary FQAN that matches one of the FQANs listed, the mapped account may receive an extra secondary GID corresponding to the matched FQAN. That GID normally is derived from the corresponding accounts in the users.conf file. If there are no accounts dedicated to that FQAN, the desired extra GID (if any) and GROUP name must be given in groups.conf. 
Note that:

- it is normal for the second and third fields to be empty, as shown in the example;
- the account corresponding to the primary FQAN does not have to belong to any secondary group: the LCMAPS library can set secondary groups independently of what is in /etc/group;
- the order of the lines in *groups.conf* is important: for any FQAN only the first match is taken

The FLAG selects a set of special accounts to be used for the mapping, namely those accounts in users.conf that have the same flag. By default, when the flag is empty, the ordinary pool accounts will be used.


## Example

### Configure users and groups mapping

We want to create the following pool accounts:
- a pool account of 100 users with group name *testvo* and *test.vo* as VO name
- a pool account of 30 production users with group name *testvoprd*, *test.vo* as VO name and *prd* as FLAG.

In order to do this we have to create the following files:

* `storm-users.conf`
* `storm-groups.conf`

The first file contains the definition of all the users (with their relative groups):

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

The second file contains the mapping rules used to map a FQAN to a local group.

```bash
"/test.vo/ROLE=R1":::prd:
"/test.vo"::::
```

In this case, we defined:
- a rule that maps *test.vo* users with role equal to `R1` to the pool account with `FLAG = prd` (group name *testvoprd*)
- a rule that maps all the other *test.vo* users to the group *testvo*

To apply the changes, define YAIM's variables into a site.def file:

```bash
USERS_CONF=/path/to/storm-users.conf
GROUPS_CONF=/path/to/storm-groups.conf
VOS=test.vo
```

and run yaim configuration as follow:

```bash
/opt/glite/yaim/bin/yaim \
    -c -s site.def \
    -n se_config_users
```

You need a YAIM profile `/opt/glite/yaim/node-info.d/se_config_users` defined as follows:

```
se_config_users_FUNCTIONS="
config_users

"
```

> Install YAIM with:
> ```
> yum install -y glite-yaim-core attr
> ```


INSTALL_ROOT USERS_CONF GROUPS_CONF VOS VO__VOMS_SERVERS GRIDMAPFILE CONFIG_GRIDMAPDIR