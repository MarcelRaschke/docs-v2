---
title: View metrics
seotitle: View metrics in InfluxDB
description: >
  View metrics about your InfluxDB system configuration and performance.
menu:
  influxdb_2_1:
    parent: Monitor InfluxDB metrics
weight: 202
related:
  - /influxdb/v2.1/monitor-alert/notification-rules/
  - /influxdb/v2.1/monitor-alert/notification-endpoints/
---

Use the InfluxDB `/metrics` API endpoint to view metrics about your InfluxDB system configuration, activity, and performance.

  - [Understand the metrics format](#understand-the-metrics-format)
  - [View all metrics](#view-all-metrics)

## Understand the metrics format

The InfluxDB `/metrics` API endpoint uses the [Prometheus text-based exposition format](https://prometheus.io/docs/instrumenting/exposition_formats) to expose InfluxDB metrics. Metrics are time-series data with the following structure:
- Data is line-oriented plain text.
- Lines are separated by a line feed character `n`.
- Each metric consists of a name in snakecase (`task_scheduler_schedule_delay`), an optional set of key-value labels enclosed in braces (`{quantile="0.5"}`), and a metric value as an integer.
- Each "metric family", a set of metrics that share the same name, is preceded by the following metadata comments:
  * *`# HELP`* `<metric family description>`
  * *`# TYPE`* `<metric family type (counter, gauge, histogram, or summary)>`

### Example

```sh
# HELP service_user_new_call_total Number of calls
# TYPE service_user_new_call_total counter
service_user_new_call_total{method="find_permission_for_user"} 9
service_user_new_call_total{method="find_user_by_id"} 4
```




## View all metrics
- Request metrics from the API
