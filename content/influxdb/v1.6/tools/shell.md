---
title: InfluxDB command line interface (CLI/shell)

menu:
  influxdb_1_6:
    name: InfluxDB command line interface (CLI/shell)
    weight: 10
    parent: Tools
---

InfluxDB's command line interface (`influx`) is an interactive shell for the HTTP API.
Use `influx` to write data (manually or from a file), query data interactively, and view query output in different formats.

* [Start `influx`](/influxdb/v1.6/tools/shell/#start-influx)
* [`influx` Arguments](/influxdb/v1.6/tools/shell/#influx-arguments)
* [`influx` Commands](/influxdb/v1.6/tools/shell/#influx-commands)

## `influx`

### Accessing the `influx` command line interface

If you [install](https://influxdata.com/downloads/) InfluxDB via a package manager, the CLI is installed at `/usr/bin/influx` (`/usr/local/bin/influx` on OS X).

To access the CLI, first start the `influxd` service and then start  `influx` in your terminal.
Once you've entered the shell and successfully connected to an InfluxDB node, you'll see the following output:

```bash
$ influx
Connected to http://localhost:8086 version 1.4.x
InfluxDB shell 1.4.x
```

> **Note:** The versions of InfluxDB and the CLI should be identical. If not, parsing issues can occur with queries.

You can now enter InfluxQL queries as well as some CLI-specific commands directly in your terminal.
You can use `help` at any time to get a list of available commands. Use `Ctrl+C` to cancel if you want to cancel a long-running InfluxQL query.

### Global options

The global options are optional arguments that can be applied when starting `influx`.

List them with `$ influx { -help | --help }`.

Optional arguments are in brackets.

#### [ `-compressed` ]

Flag indicating that the import file is compressed.
Use with `-import`.

#### [ `-consistency '{ any | one | quorum | all }'` ]

Set the write consistency level. Default value is `all`.

To see the current value, run the [`settings` command](#settings).

#### [ `-database '<db-name>'` ]

Name of the database to use when starting `influx`.

#### [ `-execute '<InfluxQL-command>'` ]

Execute an [InfluxQL](/influxdb/v1.6/query_language/data_exploration/) command and then quit.

See [Executing an InfluxQL command and quitting using `execute` ](/influxdb/v1.6/tools/shell/#executing-an-influxql-command-and-quit-with-execute).

#### [ `-format '{ json | csv | column }'` ]

Format of the server responses.

To see the current value, run the [`settings` command](#settings).

See [Specifying the server response format using `-format`](/influxdb/v1.6/tools/shell/#specify-the-format-of-the-server-responses-with-format) for an example showing the use of this option and the differences in outputs.

#### `-help`, `--help`

Displays the CLI help for usage of `influx` command line interface.

#### `-host '<hostname>'`

Host for `influx` to connect to.
Default value is `localhost`.

To see the current value, run the [`settings` command](#settings).

#### `-import`

Import new data from a file or import a previously [exported](https://github.com/influxdb/influxdb/blob/master/importer/README.md) database from a file.

The following options can be used with the `-import` option.

* `-compressed`: Flag indicating that the import file is compressed.
* `-pps '<number-of-points>'`: Throttles import to specified number of points per second.

See [Importing data from a file using `-import`](#importing-data-from-a-file-using-import) for an example.

#### `-password [ '<password>' | '' ]`

Password, used with `-username`, for connecting to the server.

To prompt for a password when `influx` starts, use empty single quotes for the password value (for example, `-password ''`).

The `influx` CLI password can also be set using the `INFLUX_PASSWORD` environment
variable.

#### `-path '<path-to-import>'`

The path to the file to import.
Use with `-import`.

#### `-port '<port>'`

The port number to which `influx` connects.
Default value is `8086`.

To see the current value, run the [`settings` command](#settings).

#### `-pps '<number-of-points>'`

Number of points per second the import allows.
Default value of `0` disables throttling of the import.
Use with `-import`.

#### `-precision '<precision-level>'`

Format, or precision, of the timestamps.
Default value is `ns` (nanoseconds).

Valid values and usage limitations:

* `rfc3339`
  - format: `YYYY-MM-DDTHH:MM:SS.nnnnnnnnnZ`
  - Supported with `-execute` command.
  - Not supported with `-import` command.
  - Example: `-precision rfc3339`
* `h` | `m` | `s` | `ms` | `u` | `ns`
  - `h` (hours), `m` (minutes), `s` (seconds), `ms` (milliseconds), or `ns` (nanoseconds)
  - Supported with `-execute` and `-import` commands.
  - Example: `-precision 'ms'`

#### `-pretty`

Enables pretty print for the `json` format.

To see the current value, run the [`settings` command](#settings).

#### `-socket '<UNIX-domain-socket>'`

UNIX domain socket to connect to.

#### `-ssl`

Use HTTPS for requests.

#### `-unsafeSsl`

Disables SSL verification when connecting to the cluster using HTTPS.

#### `-username <username> `

Username for connecting to the server. Use with `-password` option.

Alternatively, set the username for the CLI with the `INFLUX_USERNAME` environment variable.

To see the current value, run the [`settings` command](#settings).

#### `-version`

Display the InfluxDB shell version and then exit.

When starting `influx`, the server and shell versions are displayed.

### Examples

#### Executing an InfluxQL command and quitting with `-execute`

Execute queries that don't require a database specification:

```bash
$ influx -execute 'SHOW DATABASES'
name: databases
---------------
name
NOAA_water_database
_internal
telegraf
pirates
```

Execute queries that do require a database specification, and change the timestamp precision:

```bash
$ influx -execute 'SELECT * FROM "h2o_feet" LIMIT 3' -database="NOAA_water_database" -precision=rfc3339

name: h2o_feet
--------------
time			               level description	    location	     water_level
2015-08-18T00:00:00Z	 below 3 feet		        santa_monica	 2.064
2015-08-18T00:00:00Z	 between 6 and 9 feet  coyote_creek  8.12
2015-08-18T00:06:00Z	 between 6 and 9 feet  coyote_creek  8.005
```

#### Specifying server response format using `-format`

The default format is `column`:

```bash
$ influx -format=column
[...]
> SHOW DATABASES
name: databases
---------------
name
NOAA_water_database
_internal
telegraf
pirates
```

Change the format to `csv`:

```bash
$ influx -format=csv
[...]
> SHOW DATABASES
name,name
databases,NOAA_water_database
databases,_internal
databases,telegraf
databases,pirates
```

Change the format to `json`:

```bash
$ influx -format=json
[...]
> SHOW DATABASES
{"results":[{"series":[{"name":"databases","columns":["name"],"values":[["NOAA_water_database"],["_internal"],["telegraf"],["pirates"]]}]}]}
```

Change the format to `json` and enable pretty print:

```bash
$ influx -format=json -pretty
[...]
> SHOW DATABASES
{
    "results": [
        {
            "series": [
                {
                    "name": "databases",
                    "columns": [
                        "name"
                    ],
                    "values": [
                        [
                            "NOAA_water_database"
                        ],
                        [
                            "_internal"
                        ],
                        [
                            "telegraf"
                        ],
                        [
                            "pirates"
                        ]
                    ]
                }
            ]
        }
    ]
}
```

#### Importing data from a file using `-import`

The import file has two sections:

* **DDL (Data Definition Language)**: Contains the [InfluxQL commands](/influxdb/v1.6/administration/database_management/) for creating the relevant [database](/influxdb/v1.6/concepts/glossary/) and managing the [retention policy](/influxdb/v1.6/concepts/glossary/#retention-policy-rp).
If your database and retention policy already exist, your file can skip this section.
* **DML (Data Manipulation Language)**: Lists the relevant database and (if desired) retention policy and contains the data in [line protocol](/influxdb/v1.6/concepts/glossary/#line-protocol).

Example:

File (`datarrr.txt`):

```
# DDL
CREATE DATABASE pirates
CREATE RETENTION POLICY oneday ON pirates DURATION 1d REPLICATION 1

# DML
# CONTEXT-DATABASE: pirates
# CONTEXT-RETENTION-POLICY: oneday

treasures,captain_id=dread_pirate_roberts value=801 1439856000
treasures,captain_id=flint value=29 1439856000
treasures,captain_id=sparrow value=38 1439856000
treasures,captain_id=tetra value=47 1439856000
treasures,captain_id=crunch value=109 1439858880
```

Command:

```
$influx -import -path=datarrr.txt -precision=s
```

Results:

```
2015/12/22 12:25:06 Processed 2 commands
2015/12/22 12:25:06 Processed 5 inserts
2015/12/22 12:25:06 Failed 0 inserts
```

> **Note:** For large datasets, `influx` writes out a status message every 100,000 points.

Example status message:

```
>
    2015/08/21 14:48:01 Processed 3100000 lines.
    Time elapsed: 56.740578415s.
    Points per second (PPS): 54634
```

Things to note about `-import`:

* Allow the database to ingest points by using `-pps` to set the number of points per second allowed by the import. By default, pps is zero and `influx` does not throttle importing.
* Imports work with `.gz` files, just include `-compressed` in the command.
* Include timestamps in the data file. InfluxDB will assign the same timestamp to points without a timestamp. This can lead to unintended [overwrite behavior](/influxdb/v1.6/troubleshooting/frequently-asked-questions/#how-does-influxdb-handle-duplicate-points).
* If your data file has more than 5,000 points, it may be necessary to split that file into several files in order to write your data in batches to InfluxDB.
We recommend writing points in batches of 5,000 to 10,000 points.
Smaller batches, and more HTTP requests, will result in sub-optimal performance.
By default, the HTTP request times out after five seconds.
InfluxDB will still attempt to write the points after that time out but there will be no confirmation that they were successfully written.

> **Note:** For how to export data from InfluxDB version 0.8.9, see [Exporting from 0.8.9](https://github.com/influxdb/influxdb/blob/master/importer/README.md).

### Commands

The `influx` commands can run at your server shell prompt using the `influx -execute '<command>'` option or by running the commands directly at the CLI prompt after starting the `influx` shell.

You can run `help` at the CLI prompt to see a list of the available commands and usage information.

We provide detailed information on `insert` at the end of this section.

#### `auth`

Prompts for your username and password.
The `influx` utility uses these credentials when querying a database.

The `influx` username and password can also be set using:

* Global options
  - [`-username '<username>'`](#username-username)
  - [`-password '{ <password> | ''}'`](#password-password)
* Environment variables
  - `INFLUX_USERNAME`
  - `INFLUX_PASSWORD`

#### `chunked`

Enables or disables chunked responses from the server when issuing queries.
Default is enabled.

To see the current value, run the [`settings` command](#settings).

#### `chunk size <size>`

Sets the size of the chunked responses.
The default value is `10000`.
Setting the value to `0` resets the chunk size to the default value.

To see the current value, run the [`settings` command](#settings).

#### `clear '{ database | db | retention policy | rp }'`

Clears the current context for the [database](/influxdb/v1.6/concepts/glossary/#database) or [retention policy](/influxdb/v1.6/concepts/glossary/#retention-policy-rp).

#### `connect '<host:port>'`

Connect to a different server without exiting the shell.
By default, `influx` connects to `localhost:8086`.
If you do not specify either the host or the port, `influx` assumes the default setting for the missing attribute.

#### `consistency '<level>'`

Sets the write consistency level: `any`, `one`, `quorum`, or `all`.

#### `Ctrl+C`

Terminates the currently running query. Useful when an interactive query is taking too long to respond because it is trying to return too much data.

#### `exit` | `quit` | `Ctrl+D`

Quits the `influx` shell.

#### `format { json | csv | column }`

Specifies the format of the server responses. Valid values are `json`, `csv`, or `column`. Default value is `column`.

To see the current value, run the [`settings` command](#settings).

See  [Specifying the server response format using `-format`](/influxdb/v1.6/tools/shell/#specifying-the-server-response-format-using-format) for examples of each format.

#### `history`

Displays your command history.
To use the history while in the shell, simply use the "up" arrow.
`influx` stores your last 1,000 commands in your home directory in `.influx_history`.

#### `insert`

Write data using line protocol.
See [insert](/influxdb/v1.6/tools/shell/#writing-data-to-influxdb-using-insert).

#### `precision <format>`

Specifies the format/precision of the timestamp: `rfc3339` (`YYYY-MM-DDTHH:MM:SS.nnnnnnnnnZ`), `h` (hours), `m` (minutes), `s` (seconds), `ms` (milliseconds), `u` (microseconds), `ns` (nanoseconds).
Precision defaults to nanoseconds.

#### `pretty`

Turns on pretty print for the `json` format.

To see the current value, run the [`settings` command](#settings).

#### `settings`

Outputs the current settings for the shell including the `Host`, `Username`, `Database`, `Retention Policy`, `Pretty` status, `Chunked` status, `Chunk Size`, `Format`, and `Write Consistency`.

#### `use "<db_name>" | "<db_name>"."<rp_name>" ]`

Sets the current [database](/influxdb/v1.6/concepts/glossary/#database) and/or [retention policy](/influxdb/v1.6/concepts/glossary/#retention-policy-rp).
Once `influx` sets the current database and/or retention policy, there is no need to specify that database and/or retention policy in queries.
If you do not specify the retention policy, `influx` automatically queries the `use`d database's `DEFAULT` retention policy.

To see the current values for database and retention policy, run the [`settings` command](#settings).

### Writing data to InfluxDB using `insert`

Enter `insert` followed by the data in [line protocol](/influxdb/v1.6/concepts/glossary/#line-protocol) to write data to InfluxDB.
Use `insert into <retention policy> <line protocol>` to write data to a specific [retention policy](/influxdb/v1.6/concepts/glossary/#retention-policy-rp).

Write data to a single field in the measurement `treasures` with the tag `captain_id = pirate_king`.
`influx` automatically writes the point to the database's `DEFAULT` retention policy.
```
> INSERT treasures,captain_id=pirate_king value=2
>
```

Write the same point to the already-existing retention policy `oneday`:
```
> INSERT INTO oneday treasures,captain_id=pirate_king value=2
Using retention policy oneday
>
```

## Queries

Influx Query Language (InfluxQL) queries can be executed directly using the `influx` CLI shell prompt or by using the `influx -execute '<command>'` option.

For more information on InfluxQL, see:

* [Data exploration](/influxdb/v1.6/query_language/data_exploration/)
* [Schema exploration](/influxdb/v1.6/query_language/schema_exploration/)
* [Database management](/influxdb/v1.6/administration/database_management/)
* [Authentication and authorization](/influxdb/v1.6/administration/authentication_and_authorization/)
* [InfluxQL reference](/influxdb/v1.6/query_language/spec/)
