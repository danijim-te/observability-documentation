# observability-documentation

This document aims to be a more or less detailed explanation if the observability infra at ThousandEyes.

> **Note**
> These statements define the general consensus that is applied company wide and you should stick to them that there is not a good reason to break them.

* Repositories containing files to build docker images are usually named `docker-whatever`
* Repositories defining the deployments of a given service/app/... and, in most cases, its config are named `whatever-deployment`
* There is a centralized configs repository called [`te-configs`](https://github.com/thousandeyes/te-configs)

<br/>

## ThousandEyes observability cloud infra
The infra definition is defined in the [thousandeyes-aws-infrastructure](https://github.com/thousandeyes/thousandeyes-aws-infrastructure) repo.
Most (if not all) the ThousandEyes cloud infra is defined here.

There are two main dirs to look into when it comes to observability infra:

### [/environments](https://github.com/thousandeyes/thousandeyes-aws-infrastructure/tree/master/environments)
Legacy, moving to [`/stacks`](#stacks). If there's something new to be added, do it [here](#stacks). There is almost no observability infra defined here, but for the record, usually and conveniently the observability infra was defined under folders like `/environments/[ENV]/observability`.
However, some observability infra uses resources that are defined still here, like `efs` volumes for `prometheus`, `alertmanager` or `pushgateway`. Check it [here](https://github.com/thousandeyes/thousandeyes-aws-infrastructure/blob/2cd6bbfbc76efc39c5ba108957024b888eebc01e/environments/stg-sfo2/efs/params.auto.tfvars#L36).


### [/stacks](https://github.com/thousandeyes/thousandeyes-aws-infrastructure/tree/master/stacks)
The new way to go when it comes to define cloud infra at ThousandEyes. It's kind of inverse logic compared to `/environments`. This time the architecture definitions are grouped by, surprisingly, stacks. For example, the observability stack is defined under `/stacks/observability` with a `base` dir containing all the terraform files and a `layers` dir containing config for each environment. 



> **Warning**
>
> Bear in mind that PRs are merged by atlantis (commenting `atlantis merge`) with, at least, 1 approval when the PR affects to staging and **2 when impacts on production**.
>
> :no_entry_sign: **DO NOT MERGE ANY PR MANUALLY IN THIS REPO** :no_entry_sign:

<br/>

# Architecture
## Metrics
### Prometheus
* Explain Local and normal instance
* Methods used to configure scrape jobs: ec2 autodiscovery, puppet db, config file in deployment repo, annotations/k8s auto-discover, ...
* 

### Thanos
As a long term storage
* For how long?
* Where to check it?
* Process that moves the metrics to S3?

### S3
It's used for cold storage or metrics that have exceed the storage policy on Thanos. Metrics in S3 are stored:
* Where? Repo or config to check it?
* For how long? Where to check te retention policy and which mechanism is used to remove them permanently?

### Grafana
Metrics FrontEnd


### Pushgateway
* Where is it deployed?
* Deployment repo and argo links


### Alertmanager
* Where is it deployed?
* Deployment repo and argo links
* Alerting Rules: Operator, old method for rules, routing, notifying, levels that page and how to page, ...


<br/>

## Logs
### ElasticSearch
Long term logs database
* Where is it deployed?
* Deployment repo and argo links
* Retention policy
* Logs backups?

### Kinesis Firehose

### Kibana

### Fluentbit

### Logstash

### Fluentd

### Vector


<br/>

# TODO

## Useful Links
* Grafana, ES, Kibana, Argo, Artifact, AWS account list, ...



## Grafana
* https://grafana.stg.gcp.thousandeyes.com:30004/d/obs-overview/observability-overview?editPanel=7&orgId=1&from=now-90d&to=now
* deployment repo and argo
* match between different environments in the dropdown and the cluster
* metrics exposed
* different instances and kubernetes clusters where it is deployed
* Sync dashboards between stg and prod
* Meaningful health dashboards

## Thanos
* https://grafana.stg.gcp.thousandeyes.com:30004/d/obs-overview/observability-overview?editPanel=7&orgId=1&from=now-90d&to=now
* deployment repo and argo
* match between different environments in the dropdown and the cluster
* metrics exposed
* different instances and kubernetes clusters where it is deployed
* Meaningful health dashboards

## Prometheus
* https://grafana.stg.gcp.thousandeyes.com:30004/d/obs-overview/observability-overview?editPanel=7&orgId=1&from=now-90d&to=now
* deployment repo and argo
* match between different environments in the dropdown and the cluster
* metrics exposed
* different instances and kubernetes clusters where it is deployed
* Explain how normal and local instances work together > Graph
* Meaningful health dashboards


## What the heck is that thing of kinesis and how we use it

# Questions
* Why do we have `/stacks/observability` and `/stacks/opensearch-observability`?


# Service Improve points
* Log based alerts. By now it's not possible to generate alerts based on logs. There are several options for this:
  1. Use elasticsearch [alerting APIs](https://www.elastic.co/guide/en/kibana/current/alerting-apis.html).
  2. Use [kibana rules](https://www.elastic.co/guide/en/kibana/current/alerting-apis.html).
  3. Use some external service like [elastalert2](https://github.com/jertel/elastalert2)
  4. Use log-based metrics, what means to query ES, extract values from the response and expose them in a scrappable endpoint in prometheus format. Despite of there are several projects that do this, [this one](https://github.com/coliquio/http-json-query-exporter) seems used, actively maintained (04/01/2023).

    In our case it seems option 4 would be the easiest one to integrate in our infra: architect it, deploy, configure queries, scrape jobs and alerts. No important points in our infra needs to be modified. This being said, the maintaining effort has not been evaluated and needs to be considered as well.