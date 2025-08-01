---
title: Manage your database using InfluxQL
description: >
  Use InfluxQL to administer your InfluxDB server and work with InfluxDB databases, retention policies, series, measurements, and shards.
menu:
  influxdb_v1:
    name: Manage your database
    weight: 40
    parent: InfluxQL
aliases:
  - /influxdb/v1/query_language/database_management/
---

InfluxQL offers a full suite of administrative commands.

<table style="width:100%">
  <tr>
    <td><b>Data Management:</b></td>
    <td><b>Retention Policy Management:</br></td>
  </tr>
  <tr>
    <td><a href="#create-database">CREATE DATABASE</a></td>
    <td><a href="#create-retention-policies-with-create-retention-policy">CREATE RETENTION POLICY</a></td>
  </tr>
  <tr>
    <td><a href="#delete-a-database-with-drop-database">DROP DATABASE</a></td>
    <td><a href="#modify-retention-policies-with-alter-retention-policy">ALTER RETENTION POLICY</a></td>
  </tr>
  <tr>
    <td><a href="#drop-series-from-the-index-with-drop-series">DROP SERIES</a></td>
    <td><a href="#delete-retention-policies-with-drop-retention-policy">DROP RETENTION POLICY</a></td>
  </tr>
  <tr>
    <td><a href="#delete-series-with-delete">DELETE</a></td>
    <td></td>
  </tr>
  <tr>
    <td><a href="#delete-measurements-with-drop-measurement">DROP MEASUREMENT</a></td>
    <td></td>
  </tr>
  <tr>
    <td><a href="#delete-a-shard-with-drop-shard">DROP SHARD</a></td>
    <td></td>
  </tr>
</table>

If you're looking for `SHOW` queries (for example, `SHOW DATABASES` or `SHOW RETENTION POLICIES`), see [Schema Exploration](/influxdb/v1/query_language/explore-schema).

The examples in the sections below use the InfluxDB [Command Line Interface (CLI)](/influxdb/v1/introduction/getting-started/).
You can also execute the commands using the InfluxDB API; simply  send a `GET` request to the `/query` endpoint and include the command in the URL parameter `q`.
For more on using the InfluxDB API, see [Querying data](/influxdb/v1/guides/querying_data/).

> **Note:** When authentication is enabled, only admin users can execute most of the commands listed on this page.
> See the documentation on [authentication and authorization](/influxdb/v1/administration/authentication_and_authorization/) for more information.

## Data management

### CREATE DATABASE

Creates a new database.

#### Syntax

```sql
CREATE DATABASE <database_name> [WITH [DURATION <duration>] [REPLICATION <n>] [SHARD DURATION <duration>] [PAST LIMIT <duration>] [FUTURE LIMIT <duration>] [NAME <retention-policy-name>]]
```

#### Description of syntax

`CREATE DATABASE` requires a database [name](/influxdb/v1/troubleshooting/frequently-asked-questions/#what-words-and-characters-should-i-avoid-when-writing-data-to-influxdb).

The `WITH`, `DURATION`, `REPLICATION`, `SHARD DURATION`, `PAST LIMIT`,
`FUTURE LIMIT`, and `NAME` clauses are optional and create a single
[retention policy](/influxdb/v1/concepts/glossary/#retention-policy-rp)
associated with the created database.
If you do not specify one of the clauses after `WITH`, the relevant behavior
defaults to the `autogen` retention policy settings.
The created retention policy automatically serves as the database's default retention policy.
For more information about those clauses, see
[Retention Policy Management](/influxdb/v1/query_language/manage-database/#retention-policy-management).

A successful `CREATE DATABASE` query returns an empty result.
If you attempt to create a database that already exists, InfluxDB does nothing and does not return an error.

#### Examples

##### Create a database

```
> CREATE DATABASE "NOAA_water_database"
>
```

The query creates a database called `NOAA_water_database`.
[By default](/influxdb/v1/administration/config/#retention-autocreate), InfluxDB also creates the `autogen` retention policy and associates it with the `NOAA_water_database`.

##### Create a database with a specific retention policy

```
> CREATE DATABASE "NOAA_water_database" WITH DURATION 3d REPLICATION 1 SHARD DURATION 1h NAME "liquid"
>
```

The query creates a database called `NOAA_water_database`.
It also creates a default retention policy for `NOAA_water_database` with a `DURATION` of three days, a [replication factor](/influxdb/v1/concepts/glossary/#replication-factor) of one, a [shard group](/influxdb/v1/concepts/glossary/#shard-group) duration of one hour, and with the name `liquid`.

### Delete a database with DROP DATABASE

The `DROP DATABASE` query deletes all of the data, measurements, series, continuous queries, and retention policies from the specified database.
The query takes the following form:
```sql
DROP DATABASE <database_name>
```

Drop the database NOAA_water_database:
```bash
> DROP DATABASE "NOAA_water_database"
>
```

A successful `DROP DATABASE` query returns an empty result.
If you attempt to drop a database that does not exist, InfluxDB does not return an error.

### Drop series from the index with DROP SERIES

The `DROP SERIES` query deletes all points from a [series](/influxdb/v1/concepts/glossary/#series) in a database,
and it drops the series from the index.

The query takes the following form, where you must specify either the `FROM` clause or the `WHERE` clause:

```sql
DROP SERIES FROM <measurement_name[,measurement_name]> WHERE <tag_key>='<tag_value>'
```

Drop all series from a single measurement:

```sql
> DROP SERIES FROM "h2o_feet"
```

Drop series with a specific tag pair from a single measurement:

```sql
> DROP SERIES FROM "h2o_feet" WHERE "location" = 'santa_monica'
```

Drop all points in the series that have a specific tag pair from all measurements in the database:

```sql
> DROP SERIES WHERE "location" = 'santa_monica'
```

A successful `DROP SERIES` query returns an empty result.

### Delete series with DELETE

The `DELETE` query deletes all points from a
[series](/influxdb/v1/concepts/glossary/#series) in a database.
Unlike
[`DROP SERIES`](/influxdb/v1/query_language/manage-database/#drop-series-from-the-index-with-drop-series), `DELETE` does not drop the series from the index.

You must include either the `FROM` clause, the `WHERE` clause, or both:

```sql
DELETE FROM <measurement_name> WHERE [<tag_key>='<tag_value>'] | [<time interval>]
```

Delete all data associated with the measurement `h2o_feet`:

```sql
> DELETE FROM "h2o_feet"
```

Delete all data associated with the measurement `h2o_quality` and where the tag `randtag` equals `3`:

```sql
> DELETE FROM "h2o_quality" WHERE "randtag" = '3'
```

Delete all data in the database that occur before January 01, 2020:

```sql
> DELETE WHERE time < '2020-01-01'
```

Delete all data associated with the measurement `h2o_feet` in retention policy `one_day`:

```sql
> DELETE FROM "one_day"."h2o_feet"
```

A successful `DELETE` query returns an empty result.

Things to note about `DELETE`:

* `DELETE` supports
  [regular expressions](/enterprise_influxdb/v1/query_language/explore-data/#regular-expressions)
  in the `FROM` clause when specifying measurement names and in the `WHERE` clause
  when specifying tag values. It *does not* support regular expressions for the
  retention policy in the `FROM` clause.
  If deleting a series in a retention policy, `DELETE` requires that you define
  *only one* retention policy in the `FROM` clause.
* `DELETE` does not support [fields](/influxdb/v1/concepts/glossary/#field) in
  the `WHERE` clause.
* If you need to delete points in the future, you must specify that time period
  as `DELETE SERIES` runs for `time < now()` by default.

### Delete measurements with DROP MEASUREMENT

The `DROP MEASUREMENT` query deletes all data and series from the specified [measurement](/influxdb/v1/concepts/glossary/#measurement) and deletes the
measurement from the index.

The query takes the following form:
```sql
DROP MEASUREMENT <measurement_name>
```

Delete the measurement `h2o_feet`:
```sql
> DROP MEASUREMENT "h2o_feet"
```

> **Note:** `DROP MEASUREMENT` drops all data and series in the measurement.
It does not drop the associated continuous queries.

A successful `DROP MEASUREMENT` query returns an empty result.

{{% warn %}} Currently, InfluxDB does not support regular expressions with `DROP MEASUREMENT`.
See GitHub Issue [#4275](https://github.com/influxdb/influxdb/issues/4275) for more information.
{{% /warn %}}

### Delete a shard with DROP SHARD

The `DROP SHARD` query deletes a shard. It also drops the shard from the
[metastore](/influxdb/v1/concepts/glossary/#metastore).
The query takes the following form:
```sql
DROP SHARD <shard_id_number>
```

Delete the shard with the id `1`:
```
> DROP SHARD 1
>
```

A successful `DROP SHARD` query returns an empty result.
InfluxDB does not return an error if you attempt to drop a shard that does not
exist.

## Retention policy management

The following sections cover how to create, alter, and delete retention policies.
Note that when you create a database, InfluxDB automatically creates a retention policy named `autogen` which has infinite retention.
You may disable its auto-creation in the [configuration file](/influxdb/v1/administration/config/#metastore-settings).

### Create retention policies with CREATE RETENTION POLICY

#### Syntax

```sql
CREATE RETENTION POLICY <retention_policy_name> ON <database_name> DURATION <duration> REPLICATION <n> [SHARD DURATION <duration>] [PAST LIMIT <duration>] [FUTURE LIMIT <duration>] [DEFAULT]
```

#### Description of syntax

##### `DURATION`

- The `DURATION` clause determines how long InfluxDB keeps the data.
The `<duration>` is a [duration literal](/influxdb/v1/query_language/spec/#durations)
or `INF` (infinite).
The minimum duration for a retention policy is one hour and the maximum
duration is `INF`.

##### `REPLICATION`

- The `REPLICATION` clause determines how many independent copies of each point
are stored in the [cluster](/influxdb/v1/high_availability/clusters/).

- By default, the replication factor `n` usually equals the number of data nodes. However, if you have four or more data nodes, the default replication factor `n` is 3.

- To ensure data is immediately available for queries, set the replication factor `n` to less than or equal to the number of data nodes in the cluster.

> **Important:** If you have four or more data nodes, verify that the database replication factor is correct.

- Replication factors do not serve a purpose with single node instances.

##### `SHARD DURATION`

- Optional. The `SHARD DURATION` clause determines the time range covered by a [shard group](/influxdb/v1/concepts/glossary/#shard-group).
- The `<duration>` is a [duration literal](/influxdb/v1/query_language/spec/#durations)
and does not support an `INF` (infinite) duration.
- By default, the shard group duration is determined by the retention policy's
`DURATION`:

| Retention Policy's DURATION  | Shard Group Duration  |
|---|---|
| < 2 days  | 1 hour  |
| >= 2 days and <= 6 months  | 1 day  |
| > 6 months  | 7 days  |

The minimum allowable `SHARD GROUP DURATION` is `1h`.
If the `CREATE RETENTION POLICY` query attempts to set the `SHARD GROUP DURATION` to less than `1h` and greater than `0s`, InfluxDB automatically sets the `SHARD GROUP DURATION` to `1h`.
If the `CREATE RETENTION POLICY` query attempts to set the `SHARD GROUP DURATION` to `0s`, InfluxDB automatically sets the `SHARD GROUP DURATION` according to the default settings listed above.

See
[Shard group duration management](/influxdb/v1/concepts/schema_and_data_layout/#shard-group-duration-management)
for recommended configurations.

##### `PAST LIMIT`

The `PAST LIMIT` clause defines a time boundary before and relative to _now_
in which points written to the retention policy are accepted. If a point has a
timestamp before the specified boundary, the point is rejected and the write
request returns a partial write error.

For example, if a write request tries to write data to a retention policy with a
`PAST LIMIT 6h` and there are points in the request with timestamps older than
6 hours, those points are rejected.

##### `FUTURE LIMIT`

The `FUTURE LIMIT` clause defines a time boundary after and relative to _now_
in which points written to the retention policy are accepted. If a point has a
timestamp after the specified boundary, the point is rejected and the write
request returns a partial write error.

For example, if a write request tries to write data to a retention policy with a
`FUTURE LIMIT 6h` and there are points in the request with future timestamps 
greater than 6 hours from now, those points are rejected.

##### `DEFAULT`

Sets the new retention policy as the default retention policy for the database.
This setting is optional.

#### Examples

##### Create a retention policy

```
> CREATE RETENTION POLICY "one_day_only" ON "NOAA_water_database" DURATION 1d REPLICATION 1
>
```
The query creates a retention policy called `one_day_only` for the database
`NOAA_water_database` with a one day duration and a replication factor of one.

##### Create a DEFAULT retention policy

```sql
> CREATE RETENTION POLICY "one_day_only" ON "NOAA_water_database" DURATION 23h60m REPLICATION 1 DEFAULT
>
```

The query creates the same retention policy as the one in the example above, but
sets it as the default retention policy for the database.

A successful `CREATE RETENTION POLICY` query returns an empty response.
If you attempt to create a retention policy identical to one that already exists, InfluxDB does not return an error.
If you attempt to create a retention policy with the same name as an existing retention policy but with differing attributes, InfluxDB returns an error.

> **Note:** You can also specify a new retention policy in the `CREATE DATABASE` query.
See [Create a database with CREATE DATABASE](/influxdb/v1/query_language/manage-database/#create-database).

### Modify retention policies with ALTER RETENTION POLICY

The `ALTER RETENTION POLICY` query takes the following form, where you must declare at least one of the retention policy attributes `DURATION`, `REPLICATION`, `SHARD DURATION`, or `DEFAULT`:
```sql
ALTER RETENTION POLICY <retention_policy_name> ON <database_name> [DURATION <duration>] [REPLICATION <n>] [SHARD DURATION <duration>] [DEFAULT]
```

{{% warn %}} Replication factors do not serve a purpose with single node instances.
{{% /warn %}}

First, create the retention policy `what_is_time` with a `DURATION` of two days:
```sql
> CREATE RETENTION POLICY "what_is_time" ON "NOAA_water_database" DURATION 2d REPLICATION 1
>
```

Modify `what_is_time` to have a three week `DURATION`, a two hour shard group duration, and make it the `DEFAULT` retention policy for `NOAA_water_database`.
```sql
> ALTER RETENTION POLICY "what_is_time" ON "NOAA_water_database" DURATION 3w SHARD DURATION 2h DEFAULT
>
```
In the last example, `what_is_time` retains its original replication factor of 1.

A successful `ALTER RETENTION POLICY` query returns an empty result.

### Delete retention policies with DROP RETENTION POLICY

Delete all measurements and data in a specific retention policy:

{{% warn %}}
Dropping a retention policy will permanently delete all measurements and data stored in the retention policy.
{{% /warn %}}

```sql
DROP RETENTION POLICY <retention_policy_name> ON <database_name>
```

Delete the retention policy `what_is_time` in the `NOAA_water_database` database:
```bash
> DROP RETENTION POLICY "what_is_time" ON "NOAA_water_database"
>
```

A successful `DROP RETENTION POLICY` query returns an empty result.
If you attempt to drop a retention policy that does not exist, InfluxDB does not return an error.
