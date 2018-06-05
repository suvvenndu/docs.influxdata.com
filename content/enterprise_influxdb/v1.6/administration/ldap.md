---
title: Configuring LDAP authentication for InfluxDB Enterprise
description: Configuring LDAP authentication for InfluxDB Enterprise, including steps for meta nodes and data nodes, and testing LDAP connectivity.
menu:
  enterprise_influxdb_1_6:
    name: Configuring LDAP authentication
    weight: 40
    parent: Administration
---

## LDAP Preview

InfluxDB Enterprise can be configured to query a Lightweight Directory Access Protocol(LDAP)-compatible directory service to determine user permissions and to synchronize a directory service into InfluxDB so that the remote directory service does not need to be queried for each request.

## Requirements

To use LDAP with InfluxDB Enterprise, you need to support the following requirements:

* All users are managed in the remote LDAP service.


## Configuring the InfluxDB Enterprise configuration file

To use LDAP with InfluxDB Enterprise, make the following changes to the InfluxDD Enterprise configuration:

* Provide an HTTP Basic Authentication header. See [Authentication and authorization in InfluxDB](/influxdb/v1.6/administration/authentication_and_authorization/) for details on using HTTP Basic Authentication with InfluxDB.
* Provide a username and password using the following HTTP query parameters
  - `u`: username
  - `p`: password
* Enable HTTP authentication.
  - Set the environment variable `INFLUXDB_HTTP_AUTH_ENABLED`, or the corresponding `[http]` setting `auth-enabled`, to `true`. Default is `false`.
* Configure the HTTP shared secret to validate requests using JSON web tokens (JWT) and sign each HTTP payload with the secret and username.
  - Set `INFLUXDB_HTTP_SHARED_SECRET`, or the corresponding `[http]` setting `shared-secret`, to `"<shared_secret>""` (your shared secret value). Default value is `""`.


## Configuring LDAP support in InfluxDB Enterprise

Typically, database operators, and not database clients, interact directly with the meta nodes.

To use LDAP with InfluxDB Enterprise, you must:

1. Set `ldap-allowed` to `true` in each meta node configuration.

* `INFLUXDB_LDAP_ALLOWED=true`

2. Configure the ldap configuration file (`ldap.conf`).
  - `meta/ldap.md`
* sample-ldap-config.toml - point to this as a sample in `/etc` directory

[`[servers]]`
* `host` - LDAP host
* `port` - LDAP port
* Credentials to configure
  - `bind-dn` is the credential used to authenticate against LDAP
  - `bind-password` is the password
- Search filters
  - `search-base-dn`: Base DN to use for discovering an LDAP user
  - `search-filter`: LDAP filter to discover a user's DN.
  - `group-search-base-dns`(?): Base DN to use for discovering
  - `group-membership-search-filter`: LDAP filter to identify groups that a user belongs to.
  - `group-attribute`
  - `admin-groups`: List of AGs (admin groups) that are mapped to administrative privileges.
  - `group-mappings`: Mappings of the AG groups to the InfluxDB roles, which need to be created and given permissions

2. Confirm LDAP connectivity using the `ldapsearch`.
3. Verfiy using `influxd-ctl ldap ...`
* To check LDAP connectivity with given config and credentials, run the following command:
  influxd-ctl ldap verify -ldap-config /path/to/ldap.conf <usernmame> <password>


3. Load the LDAP configuration.

```
influxd-ctl ldap set-config <path-to-ldap-conf>
```

```
influxd-ctl ldap get-config
```

Run this from the meta node with the LDAP configuration.

Telegraf and Kapacitor

* Telegraf
  - telegraf.conf
    - [outputs.influxdb]
      - username
      - password
    - telegraf.d
      -
      - username
      - password
* Kapacitor
  - kapacitor.conf
    - [[influxdb]]
      - username
      - password
  -

## Confirming LDAP connectivity


After you have configured your LDAP connectivity for InfluxDB Enterprise, you can test the connectivity by following these steps:

### To confirm your LDAP connectivity

You can confirm your LDAP connectivity to InfluxDB Enterprise by running a quick test using `ldapsearch` to confirm your conne.

```
ldapsearch -p 3389 -w <password> -x -D "CN=readonly admin,OU=Users,OU=<organization_unit>,DC=<domain_controller>, ...DC=<domain_controller>,DC=com" -h <host> -b 'OU=<organization_unit>,DC=<domain_controller>,DC=<domain_controller>,DC=com' '(&(CN=<group_name>)(&(objectClass=group)))' 'member'
```
