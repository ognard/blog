+++
title = 'Learning the basics of Osquery'
date = 2024-08-05T12:54:24+02:00
slug = "learn-osquery-basics"
+++

![Osquery logo](/osquery-logo.png)

Osquery is an SQL-powered framework that serves for operating system instrumentation, monitoring, and analytics. It's open-source and was created by Facebook in 2014.

Basically, this framework allows us to use SQL queries to return various data, such as lists of the OS running processes, user accounts created on the host, processes of communication between the OS and certain suspicious domains, etc.

It is widely used by Security Analysts, Incident Responders, Threat Hunters, and so on.

Osquery is available on multiple platforms - Windows, Linux, macOS, and FreeBSD.

Once you have Osquery installed on your system, you can start using it in your terminal by running `osqueryi` command. Once started, to understand more about the functionality of the tool, you can start by running `.help` command in the terminal. For example:


```cmd
ognard@localhost$ osqueryi
Using a virtual database. Need help, type '.help'
osquery> .help
Welcome to the osquery shell. Please explore your OS!
You are connected to a transient 'in-memory' virtual database.

.all [TABLE]     Select all from a table
.bail ON|OFF     Stop after hitting an error
.connect PATH    Connect to an osquery extension socket
.disconnect      Disconnect from a connected extension socket
.echo ON|OFF     Turn command echo on or off
.exit            Exit this program
.features        List osquery's features and their statuses
.headers ON|OFF  Turn display of headers on or off
.help            Show this message
.mode MODE       Set output mode where MODE is one of:
                   csv      Comma-separated values
                   column   Left-aligned columns see .width
                   line     One value per line
                   list     Values delimited by .separator string
                   pretty   Pretty printed SQL results (default)
.nullvalue STR   Use STRING in place of NULL values
.print STR...    Print literal STRING
.quit            Exit this program
.schema [TABLE]  Show the CREATE statements
.separator STR   Change separator used by output mode
.socket          Show the local osquery extensions socket path
.show            Show the current values for various settings
.summary         Alias for the show meta command
.tables [TABLE]  List names of tables
.types [SQL]     Show result of getQueryColumns for the given query
.width [NUM1]+   Set column widths for "column" mode
.timer ON|OFF      Turn the CPU timer measurement on or off
```

## Listing tables
---

To view all of the available tables that can be queried and inspected, you can use the `.tables` command. 

```cmd
Using a virtual database. Need help, type '.help'
osquery> .tables
  => appcompat_shims
  => arp_cache
  => atom_packages
  => authenticode
  => autoexec
  => azure_instance_metadata
  => azure_instance_tags
  => background_activities_moderator
  => bitlocker_info
  => carbon_black_info
  => carves
  => certificates
  => chassis_info
  => chocolatey_packages
```

For instance, if you want to check the tables associated with processes, you can use the command `.tables processes`:

```cmd
Using a virtual database. Need help, type '.help'
osquery> .tables processes
  => appcompat_shims
  => arp_cache
  => atom_packages
  => authenticode
  => autoexec
  => azure_instance_metadata
  => azure_instance_tags
  => background_activities_moderator
  => bitlocker_info
  => carbon_black_info
  => carves
  => certificates
  => chassis_info
  => chocolatey_packages
```

or, if you want to check all of the tables associated to users, you can use `.tables user` and this will show all of the tables that contain data related to users on the OS:

```cmd
osquery> .tables user
  => user_groups
  => user_ssh_keys
  => userassist
  => users
```


## Table Schema
---

Now, in order to find out what information is contained within a table, or in other words, to find the columns of a table, you can use the command `.schema table_name`:

```cmd
osquery> .schema users
CREATE TABLE users(`uid` BIGINT, `gid` BIGINT, `uid_signed` BIGINT, `gid_signed` BIGINT, `username` TEXT, `description` TEXT, `directory` TEXT, `shell` TEXT, `uuid` TEXT, `type` TEXT, `is_hidden` INTEGER HIDDEN, `pid_with_namespace` INTEGER HIDDEN, PRIMARY KEY (`uid`, `username`, `uuid`, `pid_with_namespace`)) WITHOUT ROWID;
```

This example above, provides the `schema` of the table `users`, and with that, you have the information for all of the available columns for the `users` table. After this, you can use SQL query to display the data from particular columns. For example:

```cmd
osquery> SELECT username FROM users;

+--------------------+
| username           |
+--------------------+
| Administrator      |
| Guest              |
| John               |
| SYSTEM             |
| LOCAL SERVICE      |
| NETWORK SERVICE    |
+--------------------+
```

## Schema Documentation
---
Schema documentation is available at [this link](https://osquery.io/schema/5.12.1). This is a very useful resource where you can filter and preview the information for all of the available tables based on the Osquery version and OS. There is detailed information for every table and its columns.


## SQL queries
---

The SQL language used for querying in Osquery is not the full SQL language that you may be familiar with but is a superset of SQLite.

Basically, your querying will always start with SELECT. UPDATE and DELETE are not possible, except in situations where you may create run-time tables (views) or if you are using an extension that supports updating or deleting data.

Other than SELECT, the following are also available:

- FROM
- LIMIT
- COUNT
- JOIN
- WHERE
- LIKE
- BETWEEN

> Check out [the documentation](https://osquery.readthedocs.io/en/stable/introduction/sql/) for more information on querying in Osquery.

## Example queries
---

### COUNT

```cmd
osquery> SELECT COUNT(*) FROM users;
+----------+
| COUNT(*) |
+----------+
| 6        |
+----------+
```

### WHERE

```cmd
osquery> SELECT uid, gid, username, description FROM users WHERE username="John";

+------+-----+----------+-------------------+
| uid  | gid | username | description       |
+------+-----+----------+-------------------+
| 1009 | 544 | John     | Backend Developer |
+------+-----+----------+-------------------+
```

### JOIN

```cmd
osquery> SELECT p.pid, p.name, u.username FROM processes p JOIN users u ON u.uid=p.uid LIMIT 10;

+------+-------------------------+----------+
| pid  | name                    | username |
+------+-------------------------+----------+
| 1576 | rdpclip.exe             | James    |
| 4156 | svchost.exe             | James    |
| 4560 | svchost.exe             | James    |
| 3912 | taskhostw.exe           | James    |
| 2616 | sihost.exe              | James    |
| 5136 | ctfmon.exe              | James    |
| 5372 | explorer.exe            | James    |
| 5660 | ShellExperienceHost.exe | James    |
| 5784 | SearchUI.exe            | James    |
| 5812 | RuntimeBroker.exe       | James    |
+------+-------------------------+----------+
```