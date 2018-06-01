---
title: Database management using InfluxQL
description: Use InfluxQL administrative commands for InfluxDB databases, retention policies, series, measurements, and shards.
aliases:
  - /influxdb/v1.6/query_language/database_management/
menu:
  influxdb_1_6:
    name: Data management
    weight: 30
    parent: Administration
---

InfluxQL offers a full suite of administrative commands.

<table style="width:100%">
  <tr>
    <td><b>Data management</b></td>
    <td><b>Retention policy management</br></td>
  </tr>
  <tr>
    <td><a href="#creating-databases-using-create-database">CREATE DATABASE</a></td>
    <td><a href="#creating-retention-policies-using-create-retention-policy">CREATE RETENTION POLICY</a></td>
  </tr>
  <tr>
    <td><a href="#deleting-databases-using-drop-database">DROP DATABASE</a></td>
    <td><a href="#modifying-retention-policies-using-alter-retention-policy">ALTER RETENTION POLICY</a></td>
  </tr>
  <tr>
    <td><a href="#dropping-series-using-drop-series">DROP SERIES</a></td>
    <td><a href="#deleting-retention-policies-using-drop-retention-policy">DROP RETENTION POLICY</a></td>
  </tr>
  <tr>
    <td><a href="#deleting-series-using-delete">DELETE</a></td>
    <td></td>
  </tr>
  <tr>
    <td><a href="#deleting-measurements-using-drop-measurement">DROP MEASUREMENT</a></td>
    <td></td>
  </tr>
  <tr>
    <td><a href="#deleting-shards-using-drop-shard">DROP SHARD</a></td>
    <td></td>
  </tr>
</table>

If you're looking for `SHOW` queries (for example, `SHOW DATABASES` or `SHOW RETENTION POLICIES`), see [Schema exploration using InfluxQL](/influxdb/v1.6/query_language/schema_exploration).

The examples below use InfluxDB's [command line interface (CLI)](/influxdb/v1.6/introduction/getting-started/).
You can also execute the commands using the HTTP API; simply  send a `GET` request to the `/query` endpoint and include the command in the URL parameter `q`.
See the [Querying data with the HTTP API](/influxdb/v1.6/guides/querying_data/) for more on using the HTTP API.

> **Note:** When authentication is enabled, only admin users can execute most of the commands listed on this page.
See the documentation on [authentication and authorization](/influxdb/v1.6/administration/authentication_and_authorization/) for more information.

## Data management

### Creating databases (using `CREATE DATABASE`)

You can create a database using the `CREATE DATABASE` command.

#### Syntax and usage

```sql
CREATE DATABASE <db_name> [WITH [DURATION <duration>] [REPLICATION <n>] [SHARD DURATION <duration>] [NAME <rp_name>]]
```

`CREATE DATABASE` requires a database [name](/influxdb/v1.6/troubleshooting/frequently-asked-questions/#what-words-and-characters-should-i-avoid-when-writing-data-to-influxdb).

The `WITH`, `DURATION`, `REPLICATION`, `SHARD DURATION`, and `NAME` clauses are optional and create a single [retention policy](/influxdb/v1.6/concepts/glossary/#retention-policy-rp) associated with the created database.
If you do not specify one of the clauses after `WITH`, the relevant behavior defaults to the `autogen` retention policy settings.
The created retention policy automatically serves as the database's default retention policy.
For more information about those clauses, see [Retention Policy Management](/influxdb/v1.6/query_language/database_management/#retention-policy-management).

A successful `CREATE DATABASE` query returns an empty result.
If you attempt to create a database that already exists, InfluxDB does nothing and does not return an error.

#### Examples

##### Creating a database

The following query creates a database called `NOAA_water_database`.
[By default](/influxdb/v1.6/administration/config/#retention-autocreate-true), InfluxDB also creates the `autogen` retention policy and associates it with the `NOAA_water_database`.

```
> CREATE DATABASE "NOAA_water_database"
>
```

##### Creating a database with a specific retention policy

The following query uses the `CREATE DATABASE` command to create a database named `NOAA_water_database`.
It also creates a default retention policy for `NOAA_water_database` with a `DURATION` of three days, a [replication factor](/influxdb/v1.6/concepts/glossary/#replication-factor) of one, a [shard group](/influxdb/v1.6/concepts/glossary/#shard-group) duration of one hour, and with the name `liquid`.


```
> CREATE DATABASE "NOAA_water_database" WITH DURATION 3d REPLICATION 1 SHARD DURATION 1h NAME "liquid"
>
```

### Deleting databases (using `DROP DATABASE`)

The `DROP DATABASE` query deletes all of the data, measurements, series, continuous queries, and retention policies from the specified database.
The query takes the following form:

```sql
DROP DATABASE <database_name>
```

A successful `DROP DATABASE` query returns an empty result.
If you attempt to drop a database that does not exist, InfluxDB does not return an error.

#### Examples

Drop the database NOAA_water_database:
```bash
> DROP DATABASE "NOAA_water_database"
>
```

### Dropping series (using `DROP SERIES`)

The `DROP SERIES` statement deletes all points from a [series](/influxdb/v1.6/concepts/glossary/#series) in a database,
and it drops the series from the index.

> **Note:** `DROP SERIES` does not support time intervals in the `WHERE` clause.
See
[`DELETE`](/influxdb/v1.6/query_language/database_management/#delete-series-with-delete)
for that functionality.

The statement takes the following form, where you must specify either the `FROM` clause or the `WHERE` clause:

```sql
DROP SERIES FROM <measurement_name[,measurement_name]> WHERE <tag_key>='<tag_value>'
```

#### Dropping a series from a single measurement

```sql
> DROP SERIES FROM "h2o_feet"
```

#### Dropping a series with a specific tag pair from a single measurement

```sql
> DROP SERIES FROM "h2o_feet" WHERE "location" = 'santa_monica'
```

#### Dropping all points in a series that have a specific tag pair from all measurements in the database

```sql
> DROP SERIES WHERE "location" = 'santa_monica'
```
A successful `DROP SERIES` query returns an empty result.


### Deleting series (using `DELETE`)

The `DELETE` query deletes all points from a
[series](/influxdb/v1.6/concepts/glossary/#series) in a database.
Unlike
[`DROP SERIES`](/influxdb/v1.6/query_language/database_management/#drop-series-from-the-index-with-drop-series), it does not drop the series from the index and it supports time intervals
in the `WHERE` clause.

The query takes the following form where you must include either the `FROM`
clause or the `WHERE` clause, or both:

```
DELETE FROM <measurement_name> WHERE [<tag_key>='<tag_value>'] | [<time interval>]
```

Important points about `DELETE`:

* `DELETE` supports
[regular expressions](/influxdb/v1.6/query_language/data_exploration/#regular-expressions)
in the `FROM` clause when specifying measurement names and in the `WHERE` clause
when specifying tag values.
* `DELETE` does not support [fields](/influxdb/v1.6/concepts/glossary/#field) in the `WHERE` clause.
* If you need to delete points in the future, you must specify that time period as `DELETE SERIES` runs for `time < now()` by default. [Syntax](https://github.com/influxdata/influxdb/issues/8007)


#### Examples

##### Deleting all data associated with the measurement `h2o_feet`

```
> DELETE FROM "h2o_feet"
```

##### Deleting all data associated with the measurement `h2o_quality` and where the tag `randtag` equals `3`

```
> DELETE FROM "h2o_quality" WHERE "randtag" = '3'
```

##### Deleting all data in the database that occur before January 01, 2016

```
> DELETE WHERE time < '2016-01-01'
```
A successful `DELETE` query returns an empty result.


### Deleting measurements using DROP MEASUREMENT

The `DROP MEASUREMENT` query deletes all data and series from the specified [measurement](/influxdb/v1.6/concepts/glossary/#measurement) and deletes the
measurement from the index.

The query takes the following form:
```sql
DROP MEASUREMENT <measurement_name>
```

> **Note:** `DROP MEASUREMENT` drops all data and series in the measurement.
It does not drop the associated continuous queries.

A successful `DROP MEASUREMENT` query returns an empty result.

<dt> Currently, InfluxDB does not support regular expressions with `DROP MEASUREMENTS`.
See GitHub Issue [#4275](https://github.com/influxdb/influxdb/issues/4275) for more information.
</dt>


#### Examples

##### Deleting the measurement `h2o_feet`

```sql
> DROP MEASUREMENT "h2o_feet"
```

### Deleting shards (using `DROP SHARD`)

The `DROP SHARD` query deletes a shard. It also drops the shard from the
[metastore](/influxdb/v1.6/concepts/glossary/#metastore).
The query takes the following form:

```sql
DROP SHARD <shard_id_number>
```

#### Examples

##### Deleting the shard with the id `1`

```
> DROP SHARD 1
>
```

## Retention policy management

The following sections cover how to create, alter, and delete retention policies.
Note that when you create a database, InfluxDB automatically creates a retention policy named `autogen` which has infinite retention.
You may rename that retention policy or disable its auto-creation in the [configuration file](/influxdb/v1.6/administration/config/#metastore-settings-meta).

### Creating retention policies (using `CREATE RETENTION POLICY`)


#### Syntax and usage

```
CREATE RETENTION POLICY <retention_policy_name> ON <database_name> DURATION <duration> REPLICATION <n> [SHARD DURATION <duration>] [DEFAULT]
```
A successful `CREATE RETENTION POLICY` query returns an empty response.
If you attempt to create a retention policy identical to one that already exists, InfluxDB does not return an error.
If you attempt to create a retention policy with the same name as an existing retention policy but with differing attributes, InfluxDB returns an error.

> **Note:** You can also specify a new retention policy in the `CREATE DATABASE` query.
See [Create a database with CREATE DATABASE](/influxdb/v1.6/query_language/database_management/#create-database).

##### `DURATION`

The `DURATION` clause determines how long InfluxDB keeps the data.
The `<duration>` is a [duration literal](/influxdb/v1.6/query_language/spec/#durations)
or `INF` (infinite).
The minimum duration for a retention policy is one hour and the maximum
duration is `INF`.

##### `REPLICATION`

The `REPLICATION` clause determines how many independent copies of each point are stored in the [cluster](/influxdb/v1.6/high_availability/clusters/), where `n` is the number of data nodes.

<dt> Replication factors do not serve a purpose with single node instances.
</dt>

##### `SHARD DURATION`
<br>
The `SHARD DURATION` clause determines the time range covered by a [shard group](/influxdb/v1.6/concepts/glossary/#shard-group).
The `<duration>` is a [duration literal](/influxdb/v1.6/query_language/spec/#durations)
and does not support an `INF` (infinite) duration.
This setting is optional.
By default, the shard group duration is determined by the retention policy's `DURATION`:

| Retention Policy's DURATION  | Shard Group Duration  |
|---|---|
| < 2 days  | 1 hour  |
| >= 2 days and <= 6 months  | 1 day  |
| > 6 months  | 7 days  |

The minimum allowable `SHARD GROUP DURATION` is `1h`.
If the `CREATE RETENTION POLICY` query attempts to set the `SHARD GROUP DURATION` to less than `1h` and greater than `0s`, InfluxDB automatically sets the `SHARD GROUP DURATION` to `1h`.
If the `CREATE RETENTION POLICY` query attempts to set the `SHARD GROUP DURATION` to `0s`, InfluxDB automatically sets the `SHARD GROUP DURATION` according to the default settings listed above.

See [Shard group duration management](/influxdb/v1.6/concepts/schema_and_data_layout/#shard-group-duration-management) for recommended configurations.

##### `DEFAULT`
<br>
Sets the new retention policy as the default retention policy for the database.
This setting is optional.

#### Examples

##### Creating a retention policy

The following query creates a retention policy called `one_day_only` for the database `NOAA_water_database` with a one day duration and a replication factor of `1`.

```
> CREATE RETENTION POLICY "one_day_only" ON "NOAA_water_database" DURATION 1d REPLICATION 1
>
```

##### Creating a DEFAULT retention policy

The query creates the same retention policy as the one in the example above, but sets it as the default retention policy for the database.

```sql
> CREATE RETENTION POLICY "one_day_only" ON "NOAA_water_database" DURATION 23h60m REPLICATION 1 DEFAULT
>
```

### Modifying retention policies (using `ALTER RETENTION POLICY`)

The `ALTER RETENTION POLICY` query takes the following form, where you must declare at least one of the retention policy attributes `DURATION`, `REPLICATION`, `SHARD DURATION`, or `DEFAULT`:

```sql
ALTER RETENTION POLICY <retention_policy_name> ON <database_name> DURATION <duration> REPLICATION <n> SHARD DURATION <duration> DEFAULT
```

<dt> Replication factors do not serve a purpose with single node instances.
</dt>

#### Examples

1. Create the retention policy `what_is_time` with a `DURATION` of two days

```sql
> CREATE RETENTION POLICY "what_is_time" ON "NOAA_water_database" DURATION 2d REPLICATION 1
>
```

2. Modify `what_is_time` to have a three week `DURATION`, a 30 minute shard group duration, and  make it the `DEFAULT` retention policy for `NOAA_water_database`.

```sql
> ALTER RETENTION POLICY "what_is_time" ON "NOAA_water_database" DURATION 3w SHARD DURATION 30m DEFAULT
>
```
In the last example, `what_is_time` retains its original replication factor of 1.

A successful `ALTER RETENTION POLICY` query returns an empty result.

### Deleting retention policies (using `DROP RETENTION POLICY`)

Delete all measurements and data in a specific retention policy with:
```sql
DROP RETENTION POLICY <retention_policy_name> ON <database_name>
```

A successful `DROP RETENTION POLICY` query returns an empty result.
If you attempt to drop a retention policy that does not exist, InfluxDB does not return an error.

#### Examples

##### Deleting a retention policy in a database

In the following example, the `DROP RETENTION POLICY` command is used to delete the retention policy `what_is_time` from the database `NOAA_water_database`.

```bash
> DROP RETENTION POLICY "what_is_time" ON "NOAA_water_database"
>
```
