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
