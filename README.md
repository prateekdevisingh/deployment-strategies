# deployment-strategies
This is help to team to make sure which deployment is better for project like (POC , small project , monolithic, stateful , stateless project etc)

There are a variety of techniques to deploy new applications to production, so choosing the right strategy is an important decision, weighing the options in terms of the impact of change on the system, and on the end-users.

In this post, we are going to talk about the following strategies:

* Recreate = Version A is terminated then version B is rolled out.
* Ramped = (also known as rolling-update or incremental): Version B is slowly rolled out and replacing version A.
* Blue/Green = Version B is released alongside version A, then the traffic is switched to version B.
* Canary = Version B is released to a subset of users, then proceed to a full rollout.
* A/B testing = Version B is released to a subset of users under specific condition.
* Shadow = Version B receives real-world traffic alongside version A and doesn’t impact the response
  
![deployment strategy decision diagram](decision-diagram.png)

Before experimenting, checkout the following resources:
- [CNCF prensentation](https://www.youtube.com/watch?v=1oPhfKye5Pg)
- [CNCF prensentation slides](https://www.slideshare.net/EtienneTremel/kubernetes-deployment-strategies-cncf-webinar)
- [Kubernetes deployment strategies](https://container-solutions.com/kubernetes-deployment-strategies/)
- [Six Strategies for Application Deployment](https://thenewstack.io/deployment-strategies/).
- [Canary deployment using Istio and Helm](https://github.com/etiennetremel/istio-cross-namespace-canary-release-demo)
- [Automated rollback of Helm releases based on logs or metrics](https://container-solutions.com/automated-rollback-helm-releases-based-logs-metrics/)

## Getting started

These examples were created and tested on [Minikube](http://github.com/kubernetes/minikube)
running with Kubernetes v1.10.0.

```
$ minikube start --kubernetes-version v1.10.0 --memory 8192 --cpus 2
```


## Visualizing using Prometheus and Grafana

The following steps describe how to setup Prometheus and Grafana to visualize
the progress and performance of a deployment.

### Install Helm

To install Helm, follow the instructions provided on their
[website](https://github.com/kubernetes/helm/releases).

```
$ helm init
```

### Install Prometheus

```
$ helm install \
    --namespace=monitoring \
    --name=prometheus \
    --version=7.0.0 \
    stable/prometheus
```

### Install Grafana

```
$ helm install \
    --namespace=monitoring \
    --name=grafana \
    --version=1.12.0 \
    --set=adminUser=admin \
    --set=adminPassword=admin \
    --set=service.type=NodePort \
    stable/grafana
```

### Setup Grafana

Now that Prometheus and Grafana are up and running, you can access Grafana:

```
$ minikube service grafana
```

To login, username: `admin`, password: `admin`.

Then you need to connect Grafana to Prometheus, to do so, add a DataSource:

```
Name: prometheus
Type: Prometheus
Url: http://prometheus-server
Access: Server
```

Create a dashboard with a Graph. Use the following query:

```
sum(rate(http_requests_total{app="my-app"}[5m])) by (version)
```

To have a better overview of the version, add `{{version}}` in the legend field.

#### Example graph

Recreate:

![Kubernetes deployment recreate](recreate/grafana-recreate.png)

Ramped:

![Kubernetes deployment ramped](ramped/grafana-ramped.png)

Blue/Green:

![Kubernetes deployment blue-green](blue-green/grafana-blue-green.png)

Canary:

![Kubernetes deployment canary](canary/grafana-canary.png)

A/B testing:

![kubernetes ab-testing deployment](ab-testing/grafana-ab-testing.png)

Shadow:

![kubernetes shadow deployment](shadow/grafana-shadow.png)
