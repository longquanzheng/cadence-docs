---
layout: default
title: Cluster Monitoring
permalink: /docs/operation-guide/monitor
---

# Cluster Monitoring

## Instructions

Cadence is emitting metrics in both Server and client:

* Follow this example to emit the [client side metrics for Golang client](https://github.com/uber-common/cadence-samples/pull/36).

* Follow this example to emit the [client side metrics for Java client](https://github.com/uber/cadence-java-samples/pull/44).

* For production, follow this example to emit [server side metrics](https://github.com/uber/cadence/blob/master/config/development_prometheus.yaml#L40) to Prometheus. All services need to expose a HTTP port to provide metircs like below

```yaml
metrics:
  prometheus:
    timerType: "histogram"
    listenAddress: "127.0.0.1:8001"
```

For local server emitting metrics to Promethues, easies way is to use [docker-compose](https://github.com/uber/cadence/blob/master/docker/) to start a local Cadence.

Make sure to update the https://github.com/uber/cadence/blob/master/docker/prometheus_config.yml to ad "host.docker.internal:9098" to the scrape list before starting the docker-compose.

Note: `host.docker.internal` [may not work for some docker versions](https://docs.docker.com/docker-for-mac/networking/#use-cases-and-workarounds)

The rest of the instructions are using local environment as an example.

* After updating the prometheus_config.yaml as above, run `docker-compose up` to start the local Cadence

* Go the the sample repo, build the helloworld sample `make helloworld` and run the worker `./bin/helloworld -m worker`, and then in another Shell start a workflow `./bin/helloworld`

* Go to Prometheus server: http://localhost:9090/ , you should be able to check the metrics handler from client/frontend/matching/history/sysWorker are all healthy as targets: http://localhost:9090/targets
<img width="1192" alt="Screen Shot 2021-02-20 at 11 31 11 AM" src="https://user-images.githubusercontent.com/4523955/108606555-8d0dfb80-736f-11eb-968d-7678df37455c.png">


* Go to Grafana: http://localhost:3000 , login as admin/admin.
* Configure Prometheus as datasource: use `http://host.docker.internal:9090` as URL of prometheus.
* Import the [Grafana dashboard tempalte](/docs/operation-guide/monitor/#grafana-dashboard-templates) as JSON files.

Client side dashboard looks like this:
<img width="1513" alt="Screen Shot 2021-02-20 at 12 32 23 PM" src="https://user-images.githubusercontent.com/4523955/108607838-b7fc4d80-7377-11eb-8fd9-edc0e58afaad.png">

And server basic dashboard:
<img width="1514" alt="Screen Shot 2021-02-20 at 12 31 54 PM" src="https://user-images.githubusercontent.com/4523955/108607843-baf73e00-7377-11eb-9759-e67a1a00f442.png">


Note that some non-latency metrics are not properly shown until this fix: https://github.com/uber/cadence/pull/4007 , unless you manually configure the buckets for Prometheus.
This is because of issue https://github.com/uber/cadence/issues/4006
<img width="1519" alt="Screen Shot 2021-02-20 at 11 06 54 AM" src="https://user-images.githubusercontent.com/4523955/108606577-b169d800-736f-11eb-8fcb-88801f23b656.png">

## Grafana dashboard templates

This [package](https://github.com/uber/cadence-docs/tree/master/src/grafana) contains examples of Cadence dashboards with Prometheus.

* `Cadence-Client` is the dashboard of client metrics, and a few server side metrics that belong to client side but have to be emitted by server(for example, workflow timeout).

* `Cadence-Server-Basic` is the the basic server dashboard to monitor/navigate the health/status of a Cadence cluster.