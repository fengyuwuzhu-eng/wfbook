# 升级 Bareos

[TOC]

In most cases, a Bareos update is simply done by a package update of  the distribution. Please remind, that Bareos Director and Bareos Storage Daemon must always have the same version. The version of the File  Daemon may differ, see chapter about [backward compatibility](https://docs.bareos.org/Appendix/BackwardCompatibility.html#backward-compatibility).

## Updating the configuration files

When updating Bareos through the distribution packaging mechanism, the existing configuration kept as they are.

If you don’t want to modify the behavior, there is normally no need to modify the configuration.

However, in some rare cases, configuration changes are required. These cases are described in the [Release Notes](https://docs.bareos.org/Appendix/ReleaseNotes.html#releasenotes).

With Bareos version 16.2.4 the default configuration uses the [Subdirectory Configuration Scheme](https://docs.bareos.org/Configuration/CustomizingTheConfiguration.html#section-subdirectoryconfigurationscheme). This scheme offers various improvements. However, if your are updating  from earlier versions, your existing single configuration files (`/etc/bareos/bareos-*.conf`) stay in place and are contentiously used by Bareos. The new default configuration resource files will also be installed (`/etc/bareos/bareos-*.d/*/*.conf`). However, they will only be used, when the legacy configuration file does not exist.

See [Updates from Bareos < 16.2.4](https://docs.bareos.org/Configuration/CustomizingTheConfiguration.html#section-updatetoconfigurationsubdirectories) for details and how to migrate to [Subdirectory Configuration Scheme](https://docs.bareos.org/Configuration/CustomizingTheConfiguration.html#section-subdirectoryconfigurationscheme).

## Updating the database scheme

Sometimes improvements in Bareos make it necessary to update the database scheme.

Warning

If the Bareos catalog database does not have the current schema, the Bareos Director refuses to start.

Detailed information can then be found in the log file `/var/log/bareos/bareos.log`.

Take a look into the [Release Notes](https://docs.bareos.org/Appendix/ReleaseNotes.html#releasenotes) to see which Bareos updates do require a database scheme update.

Warning

Especially the upgrade to Bareos >= 17.2.0 restructures the **File** database table. In larger installations this is very time consuming (up to several hours or days) and temporarily doubles the amount of  required database disk space.

### Debian based Linux Distributions

Since Bareos *Version >= 14.2.0* the Debian (and Ubuntu) based packages support the **dbconfig-common** mechanism to create and update the Bareos database. If this is properly configured, the database schema will be automatically adapted by the  Bareos packages.

Warning

When using the PostgreSQL backend and updating to Bareos < 14.2.3, it is necessary to manually grant database permissions,  normally by using the following command:

```
 su - postgres -c /usr/lib/bareos/scripts/grant_bareos_privileges
```

For details see [dbconfig-common (Debian)](https://docs.bareos.org/TasksAndConcepts/CatalogMaintenance.html#section-dbconfig).

If you disabled the usage of **dbconfig-common**, follow the instructions for [Other Platforms](https://docs.bareos.org/IntroductionAndTutorial/UpdatingBareos.html#section-updatedatabaseotherdistributions).



### Other Platforms

This has to be done as database administrator. On most platforms  Bareos knows only about the credentials to access the Bareos database,  but not about the database administrator to modify the database schema.

The task of updating the database schema is done by the script **/usr/lib/bareos/scripts/update_bareos_tables**.

However, this script requires administration access to the database.  Depending on your distribution and your database, this requires  different preparations. More details can be found in chapter [Catalog Maintenance](https://docs.bareos.org/TasksAndConcepts/CatalogMaintenance.html#catmaintenancechapter).

> Warning
>
> If you’re updating to Bareos <= 13.2.3 and have  configured the Bareos database during install using Bareos environment  variables (`db_name`, `db_user` or `db_password`, see [Catalog Maintenance](https://docs.bareos.org/TasksAndConcepts/CatalogMaintenance.html#catmaintenancechapter)), make sure to have these variables defined in the same way when calling  the update and grant scripts. Newer versions of Bareos read these  variables from the Director configuration file configFileDirUnix.  However, make sure that the user running the database scripts has read  access to this file (or set the environment variables). The **postgres** user normally does not have the required permissions.

#### PostgreSQL

If your are using PostgreSQL and your PostgreSQL administrator is **postgres** (default), use following commands:

Update PostgreSQL database schema

```
su postgres -c /usr/lib/bareos/scripts/update_bareos_tables
su postgres -c /usr/lib/bareos/scripts/grant_bareos_privileges
```

The **grant_bareos_privileges** command is required, if new databases tables are introduced. It does not hurt to run it multiple times.

After this, restart the Bareos Director and verify it starts without problems.

#### MySQL/MariaDB

Make sure, that **root** has direct access to the local MySQL server. Check if the command **mysql** without parameter connects to the database. If not, you may be required to adapt your local MySQL configuration file `~/.my.cnf`. It should look similar to this:

MySQL credentials file .my.cnf

```
[client]
host=localhost
user=root
password=<input>YourPasswordForAccessingMysqlAsRoot</input>
```

If you are able to connect via the **mysql** to the database, run the following script from the Unix prompt:

Update MySQL database schema

```
/usr/lib/bareos/scripts/update_bareos_tables
```

Currently on MySQL is it not necessary to run **grant_bareos_privileges**, because access to the database is already given using wildcards.

After this, restart the Bareos Director and verify it starts without problems.







### Debian based Linux Distributions

Since Bareos *Version >= 14.2.0* the Debian (and Ubuntu) based packages support the **dbconfig-common** mechanism to create and update the Bareos database. If this is properly configured, the database schema will be automatically adapted by the  Bareos packages.

Warning

When using the PostgreSQL backend and updating to Bareos < 14.2.3, it is necessary to manually grant database permissions,  normally by using the following command:

```
 su - postgres -c /usr/lib/bareos/scripts/grant_bareos_privileges
```

For details see [dbconfig-common (Debian)](https://docs.bareos.org/TasksAndConcepts/CatalogMaintenance.html#section-dbconfig).

If you disabled the usage of **dbconfig-common**, follow the instructions for [Other Platforms](https://docs.bareos.org/IntroductionAndTutorial/UpdatingBareos.html#section-updatedatabaseotherdistributions).



### Other Platforms

This has to be done as database administrator. On most platforms  Bareos knows only about the credentials to access the Bareos database,  but not about the database administrator to modify the database schema.

The task of updating the database schema is done by the script **/usr/lib/bareos/scripts/update_bareos_tables**.

However, this script requires administration access to the database.  Depending on your distribution and your database, this requires  different preparations. More details can be found in chapter [Catalog Maintenance](https://docs.bareos.org/TasksAndConcepts/CatalogMaintenance.html#catmaintenancechapter).

> Warning
>
> If you’re updating to Bareos <= 13.2.3 and have  configured the Bareos database during install using Bareos environment  variables (`db_name`, `db_user` or `db_password`, see [Catalog Maintenance](https://docs.bareos.org/TasksAndConcepts/CatalogMaintenance.html#catmaintenancechapter)), make sure to have these variables defined in the same way when calling  the update and grant scripts. Newer versions of Bareos read these  variables from the Director configuration file configFileDirUnix.  However, make sure that the user running the database scripts has read  access to this file (or set the environment variables). The **postgres** user normally does not have the required permissions.

#### PostgreSQL

If your are using PostgreSQL and your PostgreSQL administrator is **postgres** (default), use following commands:

Update PostgreSQL database schema

```
su postgres -c /usr/lib/bareos/scripts/update_bareos_tables
su postgres -c /usr/lib/bareos/scripts/grant_bareos_privileges
```

The **grant_bareos_privileges** command is required, if new databases tables are introduced. It does not hurt to run it multiple times.

After this, restart the Bareos Director and verify it starts without problems.

#### MySQL/MariaDB

Make sure, that **root** has direct access to the local MySQL server. Check if the command **mysql** without parameter connects to the database. If not, you may be required to adapt your local MySQL configuration file `~/.my.cnf`. It should look similar to this:

MySQL credentials file .my.cnf

```
[client]
host=localhost
user=root
password=<input>YourPasswordForAccessingMysqlAsRoot</input>
```

If you are able to connect via the **mysql** to the database, run the following script from the Unix prompt:

Update MySQL database schema

```
/usr/lib/bareos/scripts/update_bareos_tables
```

Currently on MySQL is it not necessary to run **grant_bareos_privileges**, because access to the database is already given using wildcards.

After this, restart the Bareos Director and verify it starts without problems.