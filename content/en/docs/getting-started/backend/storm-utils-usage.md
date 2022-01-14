---
title: "Usage of StoRM utils scripts"
linkTitle: "StoRM utils usage"
weight: 0
noComment: true
description: >
  This is a set of useful scripts that query the StoRM DB in order to get/update storage areas-related informations.
---

### storm-get-space-aliases

The command `storm-get-space-aliases` retrieves the list of storage areas' space aliases of your StoRM instance.
Usage:

```bat
storm-get-space-aliases.sh [-u <db-username>] [-p <db-password>]
```

Example:

```bat
$ sh storm-get-space-aliases.sh -u storm -p storm
IGI_TOKEN
TAPE_TOKEN
TESTVO_TOKEN
```

This command doesn't change any information stored on database.

### storm-update-used-space

The command `storm-update-used-space` is used to **update** the used space information related to a specific space-alias.
Usage:

```bat
storm-update-used-space.sh [-u <db-username>] [-p <db-password>] [-a <spacetoken-alias>] [-s <used-space-size>]
```

The `used-space-size` must be expressed in **bytes**.

Example:

```bash
$ sh storm-update-used-space.sh -u storm -p storm -a TESTVO_TOKEN -s 52080
Getting space info for 'TESTVO_TOKEN' ...
  TOTAL_SIZE=4000000000, FREE_SIZE=3999957920, USED_SIZE=42080
Setting new free size as 3999947920 and new used space as 52080 ...
  Update query exited with 0
```