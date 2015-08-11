---
layout: post
title: "cAdvisor with StatsD, influxdb and Grafana"
description: ""
category:
tags: []
permalink: cadvisor-with-statsd-influxdb-and-grafana
---
# Why use cAdvisor with StatsD?
cAdvisor already have an influxDB backend and it work great, but, I would like to send data to StatsD and StatsD handle where I would like to sent my data: influxDB, Zabbix or Grafite.

Another point, I don't really love the schema created by cAdvisor because, prior to InfluxDB 0.9, with a lot of datapoint, I need to filter on some metric, or node or container and it was sub performant.

It's why I made a really simplist schema with StatsD and InfluxDB:
docker_node.container_name.metric with only one value. Yes, it's not the simplest schema for some agregation on dashboard on the first view, but I will explain how I do it with Grafana later.

# StatsD
Why I love StatsD?
