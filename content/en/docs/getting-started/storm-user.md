---
title: "StoRM user and permissions"
linkTitle: "Users and permissions"
date: 2021-09-21
weight: 2
noComment: true
description: >
  StoRM services run as user **storm**. This user is created during packages installation but it's a good practice to initialize it before.
---

### User creation

You can use the following commands to create StoRM user:

```shell
useradd -M storm
```

The option ```-M``` means 'without an home directory'.
You could also use specific user and group IDs as follows:

```shell
useradd -M storm -u 1234 -g 1234
```

In alternative, you could use [cnafsd-storm][storm-puppet-module] Puppet module by using [```storm::users```][storm-users] class as follows:

```puppet
# add default storm and edguser users and groups
include storm::users
```

```puppet
class { 'storm::users':
  # The list of defined users. In this case: storm and edguser have been defined.
  # Refer to puppetlabs/accounts Accounts::User::Hash
  # https://github.com/puppetlabs/puppetlabs-accounts/blob/main/types/user/hash.pp
  users  => {
    'edguser' => {
      'comment' => 'Edguser user',
      'groups'  => [ edguser, storm, ],
      'uid'     => '1200',
      'gid'     => '1200',
      'home'    => '/home/edguser',
    },
    'storm' => {
      'comment' => 'StoRM user',
      'groups'  => [ storm, edguser, ],
      'uid'     => '1000',
      'gid'     => '1000',
      'home'    => '/home/storm',
    },
  },
  # Any other group different from storm and edguser
  # Refer to puppetlabs/accounts Accounts::Group::Hash
  # https://github.com/puppetlabs/puppetlabs-accounts/blob/main/types/group/hash.pp
  groups => { },
}
```

Keep UIDs and GIDs aligned for StoRM users and groups on distributed deployments (i.e. when the services are installed on different machines).

### User file limits

It's recommended to raise the number of open files for *storm* user. Put these settings in */etc/security/limits.conf* or in a file contained in the */etc/security/limits.d* directory (recommended):

```
storm hard nofile 65535
storm soft nofile 65535
```

Edit the total amount of opened files as your needed.

You can also use [cnafsd-storm][storm-puppet-module] Puppet module by editing `storm::backend::storm_limit_nofile` property (default value: 65535).

### Storage Area's permissions

All the Storage Areas managed by StoRM needs to be owned by *storm* user. This means that, for example, the storage-area *test* root directory permissions 
must be:

```
drwxr-x---+  2 storm storm
```

The site administrator has to take care of it. To set the correct permissions on a storage area, you can launch the following commands (assuming that storm runs as user *storm*, which is the default):

```
chown -RL storm:storm <sa-root-directory>
chmod -R 750 <sa-root-directory>
```

Site administrator must also make traversable by other users the parent directories of each storage-area root directory (that's usually the same directory for all the storage-areas):

```
chmod o+x <sa-root-directory-parent>
```

[storm-puppet-module]: https://forge.puppet.com/cnafsd/storm
[storm-users]: https://italiangrid.github.io/storm-puppet-module/puppet_classes/storm_3A_3Ausers.html