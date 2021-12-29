---
title: Monitor InfluxDB metrics
seotitle: Monitor metrics of your InfluxDB instance.
description: >
  View metrics about your InfluxDB system configuration and performance. Use InfluxDB to scrape and visualize metrics from your InfluxDB instance.
menu:
  influxdb_2_1:
    parent: Monitor & alert
weight: 101
influxdb/v2.1/tags: [monitor, metrics, notifications, alert]
related:
  - /influxdb/v2.1/monitor-alert/notification-rules/
  - /influxdb/v2.1/monitor-alert/notification-endpoints/
---

Track metrics about the configuration and performance of your InfluxDB instance.

<!-- from Russ
Enabling/Disabling Prometheus /metrics endpoint
Securing the endpoint
Summary of elements found in the endpoint
Configuring Telegraf to scrape data from the endpoint and send it to cloud
Configuring an OSS scraper to scrape it
Configuring a Flux Task with promtheus.scrape to scrape it and store it locally
-->
{{< children >}}
