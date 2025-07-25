---
title: Install InfluxDB OSS
description: Install, start, and configure InfluxDB OSS.
menu:
  influxdb_v1:
    name: Install InfluxDB
    weight: 20
    parent: Introduction
aliases:
  - /influxdb/v1/introduction/installation/
alt_links:
  v2: /influxdb/v2/install/
---

This page provides directions for installing, starting, and configuring InfluxDB open source (OSS).

## InfluxDB OSS installation requirements

Installation of the InfluxDB package may require `root` or administrator privileges in order to complete successfully.

### InfluxDB OSS networking ports

By default, InfluxDB uses the following network ports:

- TCP port `8086` is available for client-server communication using the InfluxDB API.
- TCP port `8088` is available for the RPC service to perform back up and restore operations.
- TCP port `2003` is available for the Graphite protocol (when enabled).

In addition to the ports above, InfluxDB also offers multiple plugins that may
require [custom ports](/influxdb/v1/administration/ports/).
All port mappings can be modified through the [configuration file](/influxdb/v1/administration/config),
which is located at `/etc/influxdb/influxdb.conf` for default installations.

### Network Time Protocol (NTP)

InfluxDB uses a host's local time in UTC to assign timestamps to data and for
coordination purposes.
Use the Network Time Protocol (NTP) to synchronize time between hosts; if hosts'
clocks aren't synchronized with NTP, the timestamps on the data written to InfluxDB
can be inaccurate.

## Installing InfluxDB OSS

For users who don't want to install any software and are ready to use InfluxDB,
you may want to check out our
[managed hosted InfluxDB offering](https://cloud.influxdata.com).

{{< tabs-wrapper >}}
{{% tabs %}}
[Ubuntu & Debian](#)
[Red Hat & CentOS](#)
[SLES & openSUSE](#)
[FreeBSD/PC-BSD](#)
[macOS](#)
[Docker](#)
{{% /tabs %}}
{{% tab-content %}}
For instructions on how to install the Debian package from a file,
see the
[downloads page](https://influxdata.com/downloads/).

Debian and Ubuntu users can install the latest stable version of InfluxDB using the
`apt-get` package manager.

For Ubuntu/Debian users, add the InfluxData repository with the following commands:

{{< code-tabs-wrapper >}}
{{% code-tabs %}}
[wget](#)
[curl](#)
{{% /code-tabs %}}
{{% code-tab-content %}}
```sh
# influxdata-archive.key GPG fingerprint:
#   Primary key fingerprint: 24C9 75CB A61A 024E E1B6  3178 7C3D 5715 9FC2 F927
#   Subkey fingerprint:      9D53 9D90 D332 8DC7 D6C8  D3B9 D8FF 8E1F 7DF8 B07E
wget -q https://repos.influxdata.com/influxdata-archive.key
gpg --show-keys --with-fingerprint --with-colons ./influxdata-archive.key 2>&1 | grep -q '^fpr:\+24C975CBA61A024EE1B631787C3D57159FC2F927:$' && cat influxdata-archive.key | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/influxdata-archive.gpg > /dev/null
echo 'deb [signed-by=/etc/apt/trusted.gpg.d/influxdata-archive.gpg] https://repos.influxdata.com/debian stable main' | sudo tee /etc/apt/sources.list.d/influxdata.list
```
{{% /code-tab-content %}}

{{% code-tab-content %}}
```sh
# influxdata-archive.key GPG fingerprint:
#   Primary key fingerprint: 24C9 75CB A61A 024E E1B6  3178 7C3D 5715 9FC2 F927
#   Subkey fingerprint:      9D53 9D90 D332 8DC7 D6C8  D3B9 D8FF 8E1F 7DF8 B07E
curl --silent --location -O https://repos.influxdata.com/influxdata-archive.key
gpg --show-keys --with-fingerprint --with-colons ./influxdata-archive.key 2>&1 | grep -q '^fpr:\+24C975CBA61A024EE1B631787C3D57159FC2F927:$' && cat influxdata-archive.key | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/influxdata-archive.gpg > /dev/null
echo 'deb [signed-by=/etc/apt/trusted.gpg.d/influxdata-archive.gpg] https://repos.influxdata.com/debian stable main' | sudo tee /etc/apt/sources.list.d/influxdata.list
```
{{% /code-tab-content %}}
{{< /code-tabs-wrapper >}}

Then, install and start the InfluxDB service:

```bash
sudo apt-get update && sudo apt-get install influxdb
sudo service influxdb start
```

Or if your operating system is using systemd (Ubuntu 15.04+, Debian 8+):

```bash
sudo apt-get update && sudo apt-get install influxdb
sudo systemctl unmask influxdb.service
sudo systemctl start influxdb
```

{{% /tab-content %}}

{{% tab-content %}}

For instructions on how to install the RPM package from a file, please see the [downloads page](https://influxdata.com/downloads/).

Red Hat and CentOS users can install the latest stable version of InfluxDB using the `yum` package manager:

```bash
cat <<EOF | sudo tee /etc/yum.repos.d/influxdb.repo
[influxdb]
name = InfluxDB Repository - RHEL \$releasever
baseurl = https://repos.influxdata.com/rhel/\$releasever/\$basearch/stable
enabled = 1
gpgcheck = 1
gpgkey = https://repos.influxdata.com/influxdata-archive.key
EOF
```

Once repository is added to the `yum` configuration, install and start the InfluxDB service by running:

```bash
sudo yum install influxdb
sudo service influxdb start
```

Or if your operating system is using systemd (CentOS 7+, RHEL 7+):

```bash
sudo yum install influxdb
sudo systemctl start influxdb
```

{{% /tab-content %}}

{{% tab-content %}}

There are RPM packages provided by openSUSE Build Service for SUSE Linux users:

```bash
# add go repository
zypper ar -f obs://devel:languages:go/ go
# install latest influxdb
zypper in influxdb
```

{{% /tab-content %}}

{{% tab-content %}}

InfluxDB is part of the FreeBSD package system.
It can be installed by running:

```bash
sudo pkg install influxdb
```

The configuration file is located at `/usr/local/etc/influxd.conf` with examples in `/usr/local/etc/influxd.conf.sample`.

Start the backend by executing:

```bash
sudo service influxd onestart
```

To have InfluxDB start at system boot, add `influxd_enable="YES"` to `/etc/rc.conf`.

{{% /tab-content %}}

{{% tab-content %}}

Use [Homebrew](http://brew.sh/) to install InfluxDB on macOS:

```bash
brew update
brew install influxdb@1
```

{{% note %}}
##### Multiple versions of InfluxDB with Homebrew
Installing both InfluxDB 1.8 and InfluxDB 2.0 with Homebrew
can result in unexpected path and naming conflicts.
You can always run the desired version by specifying the full path:

```sh
$ /usr/local/opt/influxdb/bin/influxd version
InfluxDB {{< latest-patch >}} (git: none) build_date: 2021-04-01T17:55:08Z
$ /usr/local/opt/influxdb@1/bin/influxd version
InfluxDB v{{< latest-patch version="1.8" >}} (git: unknown unknown)
```

{{% /note %}}

{{% /tab-content %}}

{{% tab-content %}}

Use Docker to run InfluxDB v1 in a container.

For comprehensive Docker installation instructions, configuration options, and initialization features, see:

**[Install and run with Docker ›](/influxdb/v1/introduction/install/docker/)**

Quick start:

```bash
# Pull the latest InfluxDB v1.x image
docker pull influxdb:{{< latest-patch >}}

# Start InfluxDB with persistent storage
docker run -p 8086:8086 \
  -v $PWD/data:/var/lib/influxdb \
  influxdb:{{< latest-patch >}}
```

{{% /tab-content %}}
{{< /tabs-wrapper >}}

### Verify the authenticity of downloaded binary (optional)

For added security, follow these steps to verify the signature of your InfluxDB download with `gpg`.

(Most operating systems include the `gpg` command by default.
If `gpg` is not available, see the [GnuPG homepage](https://gnupg.org/download/) for installation instructions.)

1. Download and import InfluxData's public key:

    ```
    curl -s https://repos.influxdata.com/influxdata-archive.key | gpg --import
    ```

2. Download the signature file for the release by adding `.asc` to the download URL.
   For example:

    ```
    wget https://download.influxdata.com/influxdb/releases/influxdb-{{< latest-patch >}}-linux-amd64.tar.gz.asc
    ```

3. Verify the signature with `gpg --verify`:

    ```
    gpg --verify influxdb-{{< latest-patch >}}_linux_amd64.tar.gz.asc influxdb-{{< latest-patch >}}-linux-amd64.tar.gz
    ```

    The output from this command should include the following:

    ```
    gpg: Good signature from "InfluxData Package Signing Key <support@influxdata.com>" [unknown]
    ```


## Configuring InfluxDB OSS

The system has internal defaults for every configuration file setting.
View the default configuration settings with the `influxd config` command.

{{% note %}}
**Note:** If InfluxDB is being deployed on a publicly accessible endpoint, we strongly recommend authentication be enabled.
Otherwise the data will be publicly available to any unauthenticated user. The default settings do **NOT** enable
authentication and authorization. Further, authentication and authorization should not be solely relied upon to prevent access
and protect data from malicious actors. If additional security or compliance features are desired, InfluxDB should be run
behind a third-party service. Review the [authentication and authorization](/influxdb/v1/administration/authentication_and_authorization/)
settings.
{{% /note %}}

Most of the settings in the local configuration file
(`/etc/influxdb/influxdb.conf`) are commented out; all
commented-out settings will be determined by the internal defaults.
Any uncommented settings in the local configuration file override the
internal defaults.
Note that the local configuration file does not need to include every
configuration setting.

There are two ways to launch InfluxDB with your configuration file:

* Point the process to the correct configuration file by using the `-config`
option:

    ```bash
    influxd -config /etc/influxdb/influxdb.conf
    ```
* Set the environment variable `INFLUXDB_CONFIG_PATH` to the path of your
configuration file and start the process.
For example:

    ```
    echo $INFLUXDB_CONFIG_PATH
    /etc/influxdb/influxdb.conf

    influxd
    ```

InfluxDB first checks for the `-config` option and then for the environment
variable.

### Configuring InfluxDB with Docker

For detailed Docker configuration instructions including environment variables, configuration files, initialization options, and examples, see:

**[Install and run with Docker ›](/influxdb/v1/introduction/install/docker/)**

See the [Configuration](/influxdb/v1/administration/config/) documentation for more information.

### Data and WAL directory permissions

Make sure the directories in which data and the [write ahead log](/influxdb/v1/concepts/glossary#wal-write-ahead-log) (WAL) are stored are writable for the user running the `influxd` service.

{{% note %}}
**Note:** If the data and WAL directories are not writable, the `influxd` service will not start.
{{% /note %}}

The user running the `influxd` process should have the following permissions for
directories in the [InfluxDB file system](/influxdb/v1//concepts/file-system-layout/):

| Directory path   | Permission |
| :--------------- | ---------: |
| `influxdb/`      |        755 |
| `influxdb/data/` |        755 |
| `influxdb/meta/` |        755 |
| `influxdb/wal/`  |        700 |

Information about `data` and `wal` directory paths is available in the [Data settings](/influxdb/v1/administration/config/#data-settings) section of the [Configuring InfluxDB](/influxdb/v1/administration/config/) documentation.

## Hosting InfluxDB OSS on AWS

### Hardware requirements for InfluxDB

We recommend using two SSD volumes, using one for the `influxdb/wal` and the other for the `influxdb/data`.
Depending on your load, each volume should have around 1k-3k provisioned IOPS.
The `influxdb/data` volume should have more disk space with lower IOPS and the `influxdb/wal` volume should have less disk space with higher IOPS.

Each machine should have a minimum of 8GB RAM.

We’ve seen the best performance with the R4 class of machines, as they provide more memory than either of the C3/C4 class and the M4 class.

### Configuring InfluxDB OSS instances

This example assumes that you are using two SSD volumes and that you have mounted them appropriately.
This example also assumes that each of those volumes is mounted at `/mnt/influx` and `/mnt/db`.
For more information on how to do that see the Amazon documentation on how to [Add a Volume to Your Instance](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-attaching-volume.html).

### Configuration file
You'll have to update the configuration file appropriately for each InfluxDB instance you have.

```toml
...

[meta]
  dir = "/mnt/db/meta"
  ...

...

[data]
  dir = "/mnt/db/data"
  ...
wal-dir = "/mnt/influx/wal"
  ...

...

[hinted-handoff]
    ...
dir = "/mnt/db/hh"
    ...
```

### Authentication and Authorization
For all AWS deployments, we strongly recommend authentication be enabled. Without this, it is possible that your InfluxDB
instance may be publicly available to any unauthenticated user. The default settings do **NOT** enable
authentication and authorization. Further, authentication and authorization should not be solely relied upon to prevent access
and protect data from malicious actors. If additional security or compliance features are desired, InfluxDB should be run
behind additional services offered by AWS.
Review the [authentication and authorization](/influxdb/v1/administration/authentication_and_authorization/) settings.

### InfluxDB OSS permissions

When using non-standard directories for InfluxDB data and configurations, also be sure to set filesystem permissions correctly:

```bash
chown influxdb:influxdb /mnt/influx
chown influxdb:influxdb /mnt/db
```

For InfluxDB 1.7.6 or later, you must give owner permissions to the `init.sh` file. To do this, run the following script in your `influxdb` directory:

```sh
if [ ! -f "$STDOUT" ]; then
    mkdir -p $(dirname $STDOUT)
    chown $USER:$GROUP $(dirname $STDOUT)
 fi

 if [ ! -f "$STDERR" ]; then
    mkdir -p $(dirname $STDERR)
    chown $USER:$GROUP $(dirname $STDERR)
 fi

 # Override init script variables with DEFAULT values
 ```
