4[![codecov](https://codecov.io/gh/similarweb/finala/branch/master/graph/badge.svg)](https://codecov.io/gh/similarweb/finala)
![Lint](https://github.com/similarweb/finala/workflows/Lint/badge.svg)
![Fmt](https://github.com/similarweb/finala/workflows/Fmt/badge.svg)
[![Go Report Card](https://goreportcard.com/badge/github.com/similarweb/finala)](https://goreportcard.com/report/github.com/similarweb/finala)
[![Gitter](https://badges.gitter.im/similarweb-finala/community.svg)](https://gitter.im/similarweb-finala/community?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)
<p align="center">
    <img src="https://raw.githubusercontent.com/similarweb/finala/docs/update-readme-md/docs/images/logo.png" width="400">
</p>

---

# Finala

A resource cloud scanner that analyzes and reports about wasteful and unused resources to cut unwanted expenses.
The tool is based on yaml definitions (no code), by default configuration OR given yaml file and the report output will be saved in a given storage.
Currently we're not taking into account reservations, private pricing, etc.

## Supported Services
AWS:
Resource            | Sub Resources|Sub Resources|
--------------------| -------------|-------------|
DocumentDB          | -            |
DynamoDB            | -            |
EC2                 |              | Instances`|`ELB`|` NLB`|` ALB`|` EBS
ElasticCache        | -            |
IAM User            | -            |
Kinesis             | -            |
Lambda              | -            |
Neptune             | -            |
RDS                 | -            |
RedShift            | -            |

## Screenshots

### Dashboard
![alt Summary](https://raw.githubusercontent.com/similarweb/finala/master/docs/dashboard.png)

### Unused RDS report
![alt Resources](https://raw.githubusercontent.com/similarweb/finala/master/docs/resource.jpg)


### Notifications
![alt Slack](https://raw.githubusercontent.com/similarweb/finala/docs/update-readme-md/docs/images/slack.png)

## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. See deployment for notes on how to deploy the project on a live system.

### How To Use

Finala is built from 4 components:

* **API** - RESTful API server that receives events from the collector and serves the UI. See [example API configuration file](./configuration/api.yaml).

* **UI** - The User Interface, displays the data in a way that it'll look nice :).

* **Notifier** - Notifies notification groups with the support of multiple notifiers defined in [notifier.yaml](./configuration/notifier.yaml).
All resources that marked as "under utilized" are reported to the notification groups.
Currently we only support Slack notifier type [notifier.yaml](./configuration/notifier.yaml).
If you wish to contribute and add a new Notifier please read [How To add a new Notifier?](docs/notifiers/add-new-notifier.md)

* **Collector** - Collects and analyzes resources against their thresholds defined in [collector.yaml](./configuration/collector.yaml).
All resources that marked as "under utilized" are reported back to the API component.
You can define multiple accounts and regions in the [collector.yaml](./configuration/collector.yaml) file.

```yaml
providers:
  aws:
  - name: <ACCOUNT_NAME>
    # Environment variables will be used in case if these variables are absent
    access_key: <ACCESS_KEY>
    secret_key: <SECRET_KEY>
    session_token: "" # Optional variable, on default this variable not set
    regions:
      - <REGION>
```
We've already provided list of built-in cost-optimization `metrics`, you may modify the [collector.yaml](./configuration/collector.yaml) to suit your needs.
```yaml
rds:
    - description: Database connection count
        metrics:
        - name: DatabaseConnections
            statistic: Sum
        period: 24h 
        start_time: 168h # 24(h) * 7(d) = 168h
        constraint:
        operator: "=="
        value: 0
```

This example will mark RDS as under utilized if that RDS had **zero** connections in the last week.

### Deploy
You may use either approach in order to deploy Finala.

* Deploy on Kubernetes, see [Helm chart](https://github.com/similarweb/finala-helm) for more information.
* Run it locally with `docker-compose up`.

## Configuration samples explained

The full working example can be found here [collector.yaml](./configuration/collector.yaml).
<hr>

1. Find EC2 instances which have less that 5% CPU usage in the last week.
```yaml
ec2:
    - description: EC2 CPU utilization 
        metrics:
        - name: CPUUtilization
            statistic: Maximum
        period: 24h 
        start_time: 168h # 24h * 7d
        constraint:
        operator: "<"
        value: 5
```

2. Find RDS DB's that had zero connections in the last week.

```yaml
rds:
    - description: Database connection count
        metrics: 
        ### Start: Cloudwatch metrics ###
        - name: DatabaseConnections
            statistic: Sum
        period: 24h  
        start_time: 168h # 24h * 7d
        ### End: Cloudwatch metrics ###
        constraint:
        operator: "=="
        value: 0
```

3. Find ELB's that had zero traffic (requests) in the last week.

```yaml
elb:
    - description: Loadbalancer requests count
        ### Start: Cloudwatch metrics ###
        metrics:
        - name: RequestCount
            statistic: Sum
        period: 24h 
        start_time: 168h # 24h * 7d 
        ### End: Cloudwatch metrics ###
        constraint:
        operator: "=="
        value: 0   
```

4. Find Kinesis streams which don't have put records requests in the last week.
```yaml
      kinesis:
        - description: Total put records
          metrics:
            - name: "PutRecords.Bytes"
              statistic: Sum
            - name: "PutRecord.Bytes"
              statistic: Sum
          period: 24h 
          start_time: 168h # 24h * 7d
          constraint:
            # The go module Knetic/govaluate has a built in escaping
            # https://github.com/Knetic/govaluate#escaping-characters
            # [PutRecord.Bytes] will escape the parameter name
            formula: "[PutRecord.Bytes] + [PutRecords.Bytes]"
            operator: "=="
            value: 0
```
## Community, discussion, contribution, and support

You can reach the Finala community and developers via the following channels:
* [Gitter Community](https://gitter.im/similarweb-finala/community):
    * [finala-users](https://gitter.im/similarweb-finala/users)
    * [finala-developers](https://gitter.im/similarweb-finala/developers)


## Contributing to Finala

Thank you for your interest in contributing! Please refer to [Contribution guidelines](./CONTRIBUTING.md) for guidance.

**Running the different components**:

#### Collector

```shell
go run main.go collector -c ./configuration/collector.yaml
```

#### Notifier

```shell
go run main.go notifier -c ./configuration/notifier.yaml
```

#### API
```shell
go run main.go api -c ./configuration/api.yaml
```

#### UI

```shell
cd ui
npm run dev
```

*OR*

```shell
make build-ui
go run main.go ui -c ./configuration/ui.yaml
```

### Docker
Running all components using `docker-compose`:

```
docker-compose up
```


UI is exposed on port 8080 ([quick link](http://127.0.0.1:8080)).


## Running the tests

```
$ make test

$ make test-html
```
