# Dynatrace function for Google Cloud Platform monitoring

This is the home of `Dynatrace function for GCP monitoring` which provides the mechanism to pull [Google Cloud metrics](https://cloud.google.com/monitoring/api/metrics_gcp) into Dynatrace.
To help with function deployment you can use automation scripts available in this repo.
Maintaining its lifecycle places a burden on the operational team.


## Overview
`Dynatrace function for GCP monitoring` is a [Cloud Function](https://cloud.google.com/functions) written in Python that pulls metrics for configured services from [Cloud Monitoring API](https://cloud.google.com/monitoring/api/v3). Function execution is triggered by [Pub/Sub](https://cloud.google.com/pubsub) upon 1 minute schedule defined in [Cloud Scheduler](https://cloud.google.com/scheduler). Authentication token to query for metrics is retrived for the scope of [Service account](https://cloud.google.com/iam/docs/service-accounts) that is created during installation proccess. Once the timeseries are collected, the values are pushed to [Dynatrace Metrics API v2](https://www.dynatrace.com/support/help/dynatrace-api/environment-api/metric-v2/) using Dynatrace API token and URL to Dynatrace tenant environment stored in [Secret Manager](https://cloud.google.com/secret-manager).

In addition to metrics `Dynatrace function for GCP monitoring` is calling Service specific API's (for example Pub/Sub API). The purpose is to get properties of the instances that are not available in Monitoring API.  Particulary the function try to retrieve endpoint addresses (FQDN's, IP addresses).

![Architecture](./img/architecture.svg)

## Supported Google Cloud services
| Google Cloud service                 | Metric pulling | Pre-defined dashboards | Pre-defined alerts |
| --------------------------  | ---- | ---- | ---- |
| Google Cloud APIs           |  Y   |  N   |  N   |
| Google Cloud Function       |  Y   |  N   |  N   |
| Google Cloud SQL            |  Y   |  N   |  N   | 
| Google Cloud Datastore      |  Y   |  N   |  N   |
| Google Cloud Filestore      |  Y   |  N   |  N   |
| Google Cloud VM Instance    |  Y   |  N   |  N   |
| Google Cloud Storage        |  Y   |  N   |  N   |
| Google Cloud Load Balancing |  Y   |  N   |  N   |
| Google Cloud Pub/Sub        |  Y   |  N   |  N   |

## Quick Start
To start monitoring, You should deploy `Dynatrace function for GCP monitoring` into Your GCP project.

### Install with Google Cloud Shell

#### 1. Install prerequisites

To run installer script it's required to install `yq`. Please follow instrunctions from [mikefarah/yq GitHub project](https://github.com/mikefarah/yq)

For example, You may run following script:
```
sudo wget https://github.com/mikefarah/yq/releases/download/3.4.0/yq_linux_amd64 -O /usr/bin/yq && sudo chmod +x /usr/bin/yq
```

**Please note** `Dynatrace function for GCP monitoring` uses Cloud Scheduler that requires App Engine to be created. If you don't have App Engine enabled yet, installer script will prompt you to Create it and select region, where it will run. Reference: [Cloud Scheduler docummentation](https://cloud.google.com/scheduler/docs)

#### 2. Install `Dynatrace function for GCP monitoring`

Run the `Dynatrace function for GCP monitoring` installation script
```
wget https://raw.githubusercontent.com/dynatrace-oss/dynatrace-gcp-function/master/scripts/setup.sh -O setup.sh ; chmod a+x setup.sh ; ./setup.sh
```

Installation script will prompt for following parameters:
| Parameter   | Description                                   |
| ----------- | --------------------------------------------- |
| GCP project | Google Cloud project, where `Dynatrace function for GCP monitoring` should be deployed to. By default, current project set for gcloud CLI. |
| Function size | Amount of memory that should be assigned to the function. Possible options</br> **[s]** - small, up to 500 instances, 256 MB memory allocated to function</br> **[m]** - medium, up to 1000 instances, 512 MB memory allocated to function </br>**[l]** - large, up to 5000 instances, 2048 MB memory allocated to function</br>Please note that You will be able to adjust amount of memory after installation. |
| Dynatrace tenant URI | The URL to Your Dynatrace SaaS or Managed environemnt |
| Dynatrace API token | Dynatrace API token. You can learn how to generate token [Dynatrace API - Tokens and authentication](https://www.dynatrace.com/support/help/dynatrace-api/basics/dynatrace-api-authentication) manual. Integration requires `Ingest metrics using API V2` Token permission.



### Install with Bash
#### 1. Install prerequisites

To run installer script it's required to install `yq` and `jq`. Please follow instrunctions from [mikefarah/yq GitHub project](https://github.com/mikefarah/yq)

For example, You may run following script to install `yq`:
```
sudo wget https://github.com/mikefarah/yq/releases/download/3.4.0/yq_linux_amd64 -O /usr/bin/yq && sudo chmod +x /usr/bin/yq
```

With Ubuntu You can install `jq` using apt:
```
sudo apt-get install jq
```
Install [Google Cloud SDK](https://cloud.google.com/sdk/docs) following the instructions [Using the Google Cloud SDK installer](https://cloud.google.com/sdk/docs/downloads-interactive#linux)

Initialize Google Cloud Shell following the instructions [Initializing Cloud SDK](https://cloud.google.com/sdk/docs/initializing)


**Please note** `Dynatrace function for GCP monitoring` uses Cloud Scheduler that requires App Engine to be created. If you don't have App Engine enabled yet, installer script will prompt you to Create it and select region, where it will run. Reference: [Cloud Scheduler docummentation](https://cloud.google.com/scheduler/docs)

#### 2. Install `Dynatrace function for GCP monitoring`

Run the `Dynatrace function for GCP monitoring` installation script
```
wget https://raw.githubusercontent.com/dynatrace-oss/dynatrace-gcp-function/master/scripts/setup.sh -O setup.sh ; chmod a+x setup.sh ; ./setup.sh
```

Installation script will prompt for following parameters:
| Parameter   | Description                                   |
| ----------- | --------------------------------------------- |
| GCP project | Google Cloud project, where `Dynatrace function for GCP monitoring` should be deployed to. By default, current project set for gcloud CLI. |
| Function size | Amount of memory that should be assigned to the function. Possible options</br> **[s]** - small, up to 500 instances, 256 MB memory allocated to function</br> **[m]** - medium, up to 1000 instances, 512 MB memory allocated to function </br>**[l]** - large, up to 5000 instances, 2048 MB memory allocated to function</br>Please note that You will be able to adjust amount of memory after installation. |
| Dynatrace tenant URI | The URL to Your Dynatrace SaaS or Managed environemnt |
| Dynatrace API token | Dynatrace API token. You can learn how to generate token [Dynatrace API - Tokens and authentication](https://www.dynatrace.com/support/help/dynatrace-api/basics/dynatrace-api-authentication) manual. Integration requires `Ingest metrics using API V2` Token permission.



### Monitoring GCP instances in Dynatrace

Installation script deploys Dynatrace Dashboards for GCP services within provided Dynatrace tenant. You can browse them in Dynatrace UI.

Dashboards list:

![Dashboards list](./img/gcp_dashboards_list.png)

Sample dashboard for GCP API:

![Sample dashboard](./img/gcp_api_dashboard.png)

### Monitoring and troubleshooting `Dynatrace function for GCP monitoring`

`Dynatrace function for GCP monitoring` reports self-monitoring metrics as Google Metrics. This allow to track eventual problems with communication between the function and Dynatrace cluster. 

Self monitoring metrics:
| Metrics | Description |
| ------- | ----------- |
| MINT lines ingested | Amount of lines of metrics (data points) ingested into Dynatrace Metric INTerface for given interval. |
| Dynatrace connectivity | Status (1 -ok) indicating the connectivity between monitoring function and Dynatrace cluster. Connectivity can be broken due to wrong Dynatrace URL provided, wrong API token or network connectivity issues |
| Dynatrace failed requests count | Amount of requests that were rejected by Dynatrace Metric INTerface. The reason for failure can be unexpected data point value or reaching request quota for metric ingest |
| Dynatrace requests count | Amount of requests that were send to Dynatrace cluster.  |

Installation script deploy `dynatrace-gcp-function Self Monitoring` dashboard upon installation process to help tracking health of the solution.

![Self monitoring](./img/self_monitoring.png)


## Contributing

See [CONTRIBUTING](CONTRIBUTING.md) for details on submitting changes.

## License

`Dynatrace function for GCP monitoring` is under Apache 2.0 license. See [LICENSE](LICENSE) for details.