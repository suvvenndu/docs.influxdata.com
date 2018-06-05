---
title: Step 1 - Installing an InfluxDB Enterprise cluster
aliases:
    - /enterprise/v1.5/quickstart_installation/cluster_installation/

menu:
  enterprise_influxdb_1_6:
    weight: 10
    parent: QuickStart installation
---

InfluxDB Enterprise offers highly scalable clusters on your infrastructure
and a management UI for working with clusters.
The QuickStart Installation will get you up and running with a non-production
InfluxDB Enterprise cluster on three servers, each with a meta node and a data node. In a production environment, each node should be on a dedicated server. Follow the [Production installation](/enterprise_influxdb/v1.6/production_installation/) steps to create an InfluxDB Enterprise cluster for use in a production environment.

> **Note:** If you install InfluxDB Enterprise following the QuickStart Installation steps, you need to reinstall InfluxDB Enterprise following the Production Installation
steps to create an InfluxDB Enterprise cluster for production use.

## Setup description and requirements

### Setup description

The QuickStart installation steps set up a non-production InfluxDB Enterprise cluster by installing both a [meta node](/enterprise_influxdb/v1.6/concepts/glossary/#meta-node) and
a [data node](/enterprise_influxdb/v1.6/concepts/glossary/#data-node) on each of three servers and running both the [meta service](/enterprise_influxdb/v1.6/concepts/glossary/#meta-service)
and the [data service](/enterprise_influxdb/v1.6/concepts/glossary/#data-service) on each server.

### Requirements

#### License key or file

InfluxDB Enterprise requires a license key **OR** a license file to run.
Your license key is available at [InfluxPortal](https://portal.influxdata.com/licenses).
Contact support at the email we provided at signup to receive a license file.
License files are required only if the nodes in your cluster cannot reach
`portal.influxdata.com` on port `80` or `443`.

#### Networking

Meta nodes and data nodes communicate over ports `8088`,
`8089`, and `8091`.

For licensing purposes, meta nodes and data nodes must also be able to reach `portal.influxdata.com`
on port `80` or `443`.
If the nodes cannot reach `portal.influxdata.com` on port `80` or `443`,
you'll need to set the `license-path` setting instead of the `license-key`
setting in the meta node and data node configuration files.

#### Load balancer

InfluxDB Enterprise does not function as a load balancer.
You will need to configure your own load balancer to send client traffic to the
data nodes on port `8086` (the default port for the [HTTP API](/influxdb/v1.6/tools/api/)).

## Step 1: Modify the Hosts file (`/etc/hosts`) for each server

### 1.1. Add the hostnames and IP addresses for all three servers to the hosts file (`/etc/hosts`) on each of the three servers.

The hostnames below are representative:

```
<Server_1_IP_Address> quickstart-host-01
<Server_2_IP_Address> quickstart-host-02
<Server_3_IP_Address> quickstart-host-03
```

### 1.2. Verify the servers are resolvable.

Before proceeding with the installation, verify on each server that the other
servers are resolvable. Here is an example set of shell commands using `ping` and the
output for `quickstart-host-01`:

```
    ping -qc 1 quickstart-host-01
    ping -qc 1 quickstart-host-02
    ping -qc 1 quickstart-host-03

    > ping -qc 1 quickstart-host-01
    PING quickstart-host-01 (Server_1_IP_Address) 56(84) bytes of data.

    --- quickstart-host-01 ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 0ms
    rtt min/avg/max/mdev = 0.064/0.064/0.064/0.000 ms
```

Resolve any connectivity issues before proceeding with the installation.
A healthy cluster requires that every meta node and data node can communicate with every other meta node and data node.

## Step 2: Set up the meta nodes

Perform the following steps three times to add to add a meta node to each of your servers.

### 2.1. Download and install the meta node.


#### Ubuntu & Debian (64-bit)

```
wget https://dl.influxdata.com/enterprise/releases/influxdb-meta_1.6.0-c1.6.0_amd64.deb
sudo dpkg -i influxdb-meta_1.6.0-c1.6.0_amd64.deb
```
#### RedHat & CentOS (64-bit)]

```
wget https://dl.influxdata.com/enterprise/releases/influxdb-meta-1.6.0_c1.6.0.x86_64.rpm
sudo yum localinstall influxdb-meta-1.6.0_c1.6.0.x86_64.rpm
```

### 2.2. Edit the meta node configuration file

In the InfluxDB meta node's configuration file (`/etc/influxdb/influxdb-meta.conf`):

* Uncomment and set `hostname` to the full hostname of the meta node
* Set `license-key` in the `[enterprise]` section to the license key you received on InfluxPortal **OR** `license-path` in the `[enterprise]` section to the local path to the JSON license file you received from InfluxData.

<dt>
The `license-key` and `license-path` settings are mutually exclusive and one must remain set to the empty string.
</dt>

```
# Hostname advertised by this host for remote addresses.  This must be resolvable by all
# other nodes in the cluster
hostname="<quickstart-host-0x>" #✨

[enterprise]
  # license-key and license-path are mutually exclusive, use only one and leave the other blank
  license-key = "<your_license_key>" #✨ mutually exclusive with license-path

  # license-key and license-path are mutually exclusive, use only one and leave the other blank
  license-path = "/path/to/readable/JSON.license.file" #✨ mutually exclusive with license-key
```

> **Note:** The `hostname` in the configuration file must match the `hostname` in your server's host file (`/etc/hosts`).

### 2.3. Start the meta node service.

On `sysvinit` systems, run the following command to start the meta node service:
```
service influxdb-meta start
```

On `systemd` systems, run the following command to start the meta node service:
```
sudo systemctl start influxdb-meta
```

#### To verify the meta node service is running

Check to see that the service is running by entering:

```
ps aux | grep -v grep | grep influxdb-meta
```

You should see output similar to:

```
influxdb  3207  0.8  4.4 483000 22168 ?        Ssl  17:05   0:08 /usr/bin/influxd-meta -config /etc/influxdb/influxdb-meta.conf
```

## Step 3: Set up the data nodes

Perform the following steps to install data nodes on each of the three servers.

### 3.1. Download and install the data node.

#### Ubuntu & Debian (64-bit)

```
wget https://dl.influxdata.com/enterprise/releases/influxdb-data_1.6.0-c1.6.0_amd64.deb
sudo dpkg -i influxdb-data_1.6.0-c1.6.0_amd64.deb
```
#### RedHat & CentOS (64-bit)

```
wget https://dl.influxdata.com/enterprise/releases/influxdb-data-1.6.0_c1.6.0.x86_64.rpm
sudo yum localinstall influxdb-data-1.6.0_c1.6.0.x86_64.rpm
```

### 3.2. Edit the InfluxDB data node's configuration file.

1. In the InfluxDB data node's configuration file (`/etc/influxdb/influxdb.conf`):

* Uncomment `hostname` at the top of the file and set it to the full hostname of the data node
* Uncomment `auth-enabled` in the `[http]` section and set it to `true`
* Uncomment `shared-secret` in the `[http]` section and set it to a long pass phrase that will be used to sign tokens for intra-cluster communication. This value must be consistent across all data nodes.

> **Note:** When you enable authentication, InfluxDB only executes HTTP requests that are sent with valid credentials.
See the [authentication section](/influxdb/latest/administration/authentication_and_authorization/#authentication) for more information.

2. In the InfluxDB data node's configuration file (`/etc/influxdb/influxdb.conf`):

* Set the `license-key` in the `[enterprise]` section to the license key you received on InfluxPortal **OR** `license-path` in the `[enterprise]` section to the local path to the JSON license file you received from InfluxData.

<dt>
The `license-key` and `license-path` settings are mutually exclusive and one must remain set to the empty string.
</dt>

```
# Change this option to true to disable reporting.
# reporting-disabled = false
# bind-address = ":8088"
hostname="<quickstart-host-0x>" #✨

[enterprise]
  # license-key and license-path are mutually exclusive, use only one and leave the other blank
  license-key = "<your_license_key>" #✨ mutually exclusive with license-path

  # The path to a valid license file.  license-key and license-path are mutually exclusive,
  # use only one and leave the other blank.
  license-path = "/path/to/readable/JSON.license.file" #✨ mutually exclusive with license-key

[meta]
  # Where the cluster metadata is stored
  dir = "/var/lib/influxdb/meta" # data nodes do require a local meta directory

[...]

[http]
  # Determines whether HTTP endpoint is enabled.
  # enabled = true

  # The bind address used by the HTTP service.
  # bind-address = ":8086"

  # Determines whether HTTP authentication is enabled.
  auth-enabled = true #✨ this is recommended but not required

[...]

  # The JWT auth shared secret to validate requests using JSON web tokens.
  shared-secret = "long pass phrase used for signing tokens" #✨
```
> **Note:** The `hostname` in the configuration file must match the `hostname` in your server's `/etc/hosts` file.


### 3.3. Start the InfluxDB data node service.

On `sysvinit` systems, run the following command to start the data service:

```
service influxdb start
```

On `systemd` systems, run the following command to start the data service:

```
sudo systemctl start influxdb
```

#### Verify that the data node service is running

Check to see that the service is running by entering:

```
ps aux | grep -v grep | grep influxdb
```

You should see output similar to:

```
influxdb  2706  0.2  7.0 571008 35376 ?        Sl   15:37   0:16 /usr/bin/influxd -config /etc/influxdb/influxdb.conf
```

If you do not see the expected output, the service is either not starting or is exiting prematurely. Check the [logs](/enterprise_influxdb/v1.6/administration/logs/) for error messages and verify the previous setup steps are complete.

## Step 4: Join the nodes to the cluster.

### 4.1. Join the first server to the cluster

On the first server (`quickstart-host-01`), join its meta node and data node
to the cluster by running the following command:

```
influxd-ctl join
```

The expected output is:

```
Joining meta node at localhost:8091
Searching for meta node on quickstart-host-01:8091...
Searching for data node on quickstart-host-01:8088...

Successfully created cluster

  * Added meta node 1 at quickstart-host-01:8091
  * Added data node 2 at quickstart-host-01:8088

  To join additional nodes to this cluster, run the following command:

  influxd-ctl join quickstart-host-01:8091
```

> **Note:** The `influxd-ctl join` command has an optional flag `-v` that enables verbose information about the join. For example:

```
influxd-ctl join -v quickstart-host-01:8091
```

>To confirm that the node was successfully joined, run `influxd-ctl show` and verify that the node's hostname shows in the output.

### II. Join the second server to the cluster

On the second server (`quickstart-host-02`), join its meta node and data node
to the cluster by entering:

```
influxd-ctl join quickstart-host-01:8091
```

The expected output is:

```
Joining meta node at quickstart-host-01:8091
Searching for meta node on quickstart-host-02:8091...
Searching for data node on quickstart-host-02:8088...

Successfully joined cluster

  * Added meta node 3 at quickstart-host-02:8091
  * Added data node 4 at quickstart-host-02:8088
```

### III. Join the third server to the cluster

On the third server (`quickstart-host-03`), join its meta node and data node
to the cluster by entering:

```
influxd-ctl join quickstart-host-01:8091
```

The expected output is:

```
Joining meta node at quickstart-host-01:8091
Searching for meta node on quickstart-host-03:8091...
Searching for data node on quickstart-host-03:8088...

Successfully joined cluster

  * Added meta node 5 at quickstart-host-03:8091
  * Added data node 6 at quickstart-host-03:8088
```

### IV. Verify your cluster

On any server, enter:

```
influxd-ctl show
```

The expected output is:
```
Data Nodes
==========
ID   TCP Address                  Version
2    quickstart-host-01:8088   1.6.0-c1.6.0
4    quickstart-host-02:8088   1.6.0-c1.6.0
6    quickstart-host-03:8088   1.6.0-c1.6.0

Meta Nodes
==========
TCP Address                  Version
quickstart-host-01:8091   1.6.0-c1.6.0
quickstart-host-02:8091   1.6.0-c1.6.0
quickstart-host-03:8091   1.6.0-c1.6.0
```

Your InfluxDB Enterprise cluster should now have three data nodes and three meta nodes.
If you do not see your meta or data nodes in the output, retry
adding them to the cluster.

Once all of your nodes are joined to the cluster, move on to the [next step](/enterprise_influxdb/v1.6/quickstart_installation/chrono_install)
in the QuickStart installation to set up Chronograf.
