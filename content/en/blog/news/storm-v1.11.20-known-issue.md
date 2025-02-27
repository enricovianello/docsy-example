---
title: "StoRM v1.11.20 could break connections with MariaDB"
linkTitle: "StoRM v1.11.20 known issue"
date: 2021-04-30
description: >
  After the update from StoRM v1.11.19 to StoRM v1.11.20, if JVM and database are not on the same timezone, the Backend’s communication with MariaDB could start failing.
---

After the update from StoRM v1.11.19 to StoRM v1.11.20, if JVM and database are not on the same timezone, the Backend's communication with MariaDB could start failing with the following error:

```
Caused by: java.sql.SQLException: The server time zone value 'CEST' is unrecognized or represents more than one time zone. You must configure either the server or JDBC driver (via the serverTimezone configuration property) to use a more specifc time zone value if you want to utilize time zone support.
```

This bug is tracked at [STOR-1397](https://issues.infn.it/jira/browse/STOR-1397).

The possible solutions to avoid this problem are:
* downgrade StoRM Backend to v1.11.19 (**recommended**)
* apply a workaround within MariaDB
* install StoRM Backend v1.11.21 beta 

### Downgrade StoRM Backend to v1.11.19

**RECOMMENDED**. It's the fastest and easy thing to do. The command:

```
yum downgrade storm-backend-server
```

should ask you to install StoRM Backend v1.11.19. Confirm and restart the service:

```
systemctl restart storm-backend-server
```

### Apply a workaround within MariaDB

1. **Populate** `mysql.time_zone_name` **table**

    The `mysql.time_zone_name` table should be empty:

    ```
    MariaDB [mysql]> SELECT * FROM mysql.time_zone_name;
    Empty set (0.00 sec)
    ```

    and global variable `time_zone` should be set to SYSTEM.

    ```
    MariaDB [mysql]> SHOW GLOBAL VARIABLES LIKE 'time_zone';
    +---------------+--------+
    | Variable_name | Value  |
    +---------------+--------+
    | time_zone     | SYSTEM |
    +---------------+--------+
    1 row in set (0.00 sec)
    ```

    We need to populate that table as follows:

    ```
    mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql -uroot -pYOUR_MYSQL_ROOT_PASSWORD mysql
    ```

1. **Set** `default_time_zone`

    Edit `/etc/my.cnf.d/server.conf` and add under `[mysql]` section add the following:

    ```
    default_time_zone = Europe/Rome
    ```

    Or choose your proper `time_zone`.

3. **Restart MySQL**

    ```
    systemctl restart mariadb
    ```

    And check new time zone value:

    ```
    MariaDB [(none)]> SHOW GLOBAL VARIABLES LIKE 'time_zone';
    +---------------+-------------+
    | Variable_name | Value       |
    +---------------+-------------+
    | time_zone     | Europe/Rome |
    +---------------+-------------+
    1 row in set (0.00 sec)
    ```

4. **Restart StoRM Backend**

    ```
    systemctl restart storm-backend-server
    ```

### Install StoRM Backend v1.11.21 beta

StoRM beta repository already contains a beta version of next StoRM v1.11.21 with this error fixed.
Of course beta repository is not a stable repo, so apply this in case of a testbed. We do not recommend use this rpms in a production behaviour.

Browse CentOS 7 Beta repository [here](https://repo.cloud.cnaf.infn.it/service/rest/repository/browse/storm-rpm-beta/centos7/).
