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

# Recreate
The recreate strategy is a dummy deployment which consists of shutting down version A then deploying version B after version A is turned off. This technique implies downtime of the service that depends on both shutdown and boot duration of the application.

* Pros:

    * Easy to setup.
    * Application state entirely renewed.

* Cons:

    * High impact on the user, expect downtime that depends on both shutdown and boot duration of the application.

# Ramped
The ramped deployment strategy consists of slowly rolling out a version of an application by replacing instances one after the other until all the instances are rolled out. It usually follows the following process: with a pool of version A behind a load balancer, one instance of version B is deployed. When the service is ready to accept traffic, the instance is added to the pool. Then, one instance of version A is removed from the pool and shut down.

Depending on the system taking care of the ramped deployment, you can tweak the following parameters to increase the deployment time:

Parallelism, max batch size: Number of concurrent instances to roll out.
Max surge: How many instances to add in addition of the current amount.
Max unavailable: Number of unavailable instances during the rolling update procedure.

* Pros:

    * Easy to set up.
    * Version is slowly released across instances.
    * Convenient for stateful applications that can handle rebalancing of the data.

* Cons:

    * Rollout/rollback can take time.
    * Supporting multiple APIs is hard.
    * No control over traffic.

# Blue/Green
The blue/green deployment strategy differs from a ramped deployment, version B (green) is deployed alongside version A (blue) with exactly the same amount of instances. After testing that the new version meets all the requirements the traffic is switched from version A to version B at the load balancer level.

* Pros:

    * Instant rollout/rollback.
    * Avoid versioning issue, the entire application state is changed in one go.
* Cons:

    * Expensive as it requires double the resources.
    * Proper test of the entire platform should be done before releasing to production.
    * Handling stateful applications can be hard.

# Canary
A canary deployment consists of gradually shifting production traffic from version A to version B. Usually the traffic is split based on weight. For example, 90 percent of the requests go to version A, 10 percent go to version B.

This technique is mostly used when the tests are lacking or not reliable or if there is little confidence about the stability of the new release on the platform.
  
* Pros:

    * Version released for a subset of users.
    * Convenient for error rate and performance monitoring.
    * Fast rollback.
* Con:

    * Slow rollout.  

# A/B testing
A/B testing deployments consists of routing a subset of users to a new functionality under specific conditions. It is usually a technique for making business decisions based on statistics, rather than a deployment strategy. However, it is related and can be implemented by adding extra functionality to a canary deployment so we will briefly discuss it here.

This technique is widely used to test conversion of a given feature and only roll-out the version that converts the most.

Here is a list of conditions that can be used to distribute traffic amongst the versions:

  * By browser cookie
  * Query parameters
  * Geolocalisation
  * Technology support: browser version, screen size, operating system, etc.
  * Language

* Pros:

    * Several versions run in parallel.
    * Full control over the traffic distribution.
* Cons:

    * Requires intelligent load balancer.
    * Hard to troubleshoot errors for a given session, distributed tracing becomes mandatory.

# Shadow
A shadow deployment consists of releasing version B alongside version A, fork version A’s incoming requests and send them to version B as well without impacting production traffic. This is particularly useful to test production load on a new feature. A rollout of the application is triggered when stability and performance meet the requirements.

This technique is fairly complex to setup and needs special requirements, especially with egress traffic. For example, given a shopping cart platform, if you want to shadow test the payment service you can end-up having customers paying twice for their order. In this case, you can solve it by creating a mocking service that replicates the response from the provider.

* Pros:

    * Performance testing of the application with production traffic.
    * No impact on the user.
    * No rollout until the stability and performance of the application meet the requirements.
* Cons:

    * Expensive as it requires double the resources.
    * Not a true user test and can be misleading.
    * Complex to setup.
    * Requires mocking service for certain cases.


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
