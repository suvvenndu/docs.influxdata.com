---
title: Administering InfluxDB
description: Covers information about configuration, authentication and authorization, database management, upgrading, security issues, logging and tracing.
menu:
  influxdb_1_6:
    name: Administration
    weight: 50
---
The administration documentation contains all the information needed to administer a working InfluxDB installation.

## [Configuring InfluxDB](/influxdb/v1.6/administration/config/)

Information about the config file `influx.conf`

#### [Authentication and authorization](/influxdb/v1.6/administration/authentication_and_authorization/)

Covers how to
[set up authentication](/influxdb/v1.6/administration/authentication_and_authorization/#set-up-authentication)
and how to
[authenticate requests](/influxdb/v1.6/administration/authentication_and_authorization/#authenticate-requests) in InfluxDB.
This page also describes the different
[user types](/influxdb/v1.6/administration/authentication_and_authorization/#user-types-and-privileges) and the InfluxQL for
[managing database users](/influxdb/v1.6/administration/authentication_and_authorization/#user-management-commands).

#### [Database management](/influxdb/v1.6/administration/database_management/)

Covers using InfluxQL administrative commands for managing
[databases](/influxdb/v1.6/concepts/glossary/#database) and
[retention policies](/influxdb/v1.6/concepts/glossary/#retention-policy-rp) in InfluxDB.
See Database Management for creating and dropping databases and retention
policies as well as deleting and dropping data.

## [Upgrading](/influxdb/v1.6/administration/upgrading/)

Information about upgrading from previous versions of InfluxDB

## [Enabling HTTPS](/influxdb/v1.6/administration/https_setup/)

Enabling HTTPS to secure the communication between clients and the InfluxDB
server, and verifying the authenticity of the InfluxDB server to clients.

## [Logging and tracing in InfluxDB](/influxdb/v1.6/administration/logs/)

Information on structured logging, redirecting InfluxDB log output, and tracing for debugging and troubleshooting.

## [Ports](/influxdb/v1.6/administration/ports/)

## [Backing up and restoring](/influxdb/v1.6/administration/backup_and_restore/)

Procedures to backup data created by InfluxDB and to restore from a backup.

## [Managing security](/influxdb/v1.6/administration/security/)

Overview of security options and configurations.

## [Stability and compatibility](/influxdb/v1.6/administration/stability_and_compatibility/)

Management of breaking changes, upgrades, and ongoing support.
