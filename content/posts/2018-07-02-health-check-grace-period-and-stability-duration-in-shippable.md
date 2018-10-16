+++ 
date = 2018-07-03T06:54:42-04:00
title = "Health Check Grace Period and Stability Duration in Shippable"
slug = "health-check-grace-period-and-stability-duration-in-shippable" 
tags = ["shippable", "ci", "cd", "aws"]
categories = []
+++

# Short Story

## Use Health Check Grace Period

You want to use ECS's `healthCheckGracePeriodSeconds`, the amount of time before the ALB begins checking the container for health. This can be set in your `shippable.resources.yml` file, as a `dockerOption`:

{{< highlight yaml >}}
resources:
  - name: nameOfResource
    type: dockerOptions
    version:
      cpuShares: 768
      memory: 1024
      portMappings:
        - 0:443
      entryPoint:
        - /someEntryPoint
      privileged: false
      service:
        healthCheckGracePeriodSeconds: 60
{{< / highlight >}}

This should be set to something larger than your container's startup time, to when it can receive traffic at it's health endpoint.

## Use Shippable's "Stability Duration"

The stability duration will make sure your ECS service has the prescribed number of containers for a set amount of time before bringing the old service down. You can set this in the shippable jobs section:

{{< highlight yaml >}}
jobs:
  - name: some-deploy-name
    type: deploy
    stabilityDuration: 120
    on_success:
      - NOTIFY: some-notification
    on_failure:
      - etc etc
{{< / highlight >}}

A good rule of thumb would be `healthCheckGracePeriodSeconds` + "time for unhealthy checks" + some buffer of time.

# Long story

When using [Shippable](https://shippable.com) with AWS, it's important to understand exactly how deployment works, and what Shippable does and does not do. If you do not, you may end up in a state where your code does not deploy correctly, does not do a blue-green deployment correctly, leaving you with down time.

This post is going to assume that you have some knowledge about services in Amazon's ECS, and how it works, and already know a lot about how Shippable deployments work.

## Health Check Grace Period

First, let's understand what exactly happens when you try to bring up docker containers in AWS. This will assume you are using ALBs and Target Groups, but the same principles should apply for ELBs as well.

<img src="/images/bypost/health-check-grace-period-and-stability-duration-in-shippable/health-check-example.png" height="200">

Above is a typical configuration for a target group. What you can see from this image is that the `Unhealthy threshold` is currently set to 6 pings, with an interval of 6 seconds. 

As soon as a docker container is started (regardless of if it's ready to receive traffic), the ALB will begin pinging the health endpoint to see if it is healthy. If it reaches the unhealthy threshold, it marks the container as bad, and will bring it down. 

So, let's say that your container starts at time 0, and your first health check is at time 0. Unless your service starts up super fast, it's going to fail that first health check. This means you have 5 health checks left, spaced 6 seconds apart left. If your service is not available to respond to health checks within 30 seconds, it will mark the container as unhealthy and kill it, trying to bring up a new one.

Originally, a response to this issue would be to increase the `Unhealthy Threshold` to something greater, so that the container has enough time to initialize. This comes at a cost, however. If the container is in a normal state, then fails for whatever reason, it will not be removed from receiving requests until the unhealthy threshold is hit. Let's say for example you increased it to 11 (giving it another 30 seconds). In normal operations, this means that your container could be jacked up for a whole minute before the ALB will stop sending requests to it, and ECS will try to replace it with a new one.

However, in December 2017 [Amazon released the "health check grace period" for containers](https://aws.amazon.com/about-aws/whats-new/2017/12/amazon-ecs-adds-elb-health-check-grace-period/). This means that ECS won't begin checking the health of the docker container until after this time has passed. If you set this time to greater than the start up time of your container, you'll then be able to set your `Unhealthy threshold` to whatever you feel is appropriate, rather than tailoring it to the startup time of your container.

## Stability Duration

It's important to understand how Shippable does blue-green deployments. In general, you may believe a blue-green deployment is suppose to bring up a new ECS service (blue), ensures that it is stable, and then will bring the down the original service (green) once the new one is up.

The way Shippable determines that the blue service is stable, however, is very unsophisticated. It simply looks at the number of containers that are suppose to be running against the number of containers that are running in the service. It does not know about the health of the containers, or if they will continue to stay up.

So, to give an example, let's say your ECS service has 2 containers it needs to run, but they take 60 seconds to come up. Once shippable has started a deployment, it is just looking for two containers in the new service, which may happen 5 seconds after creation. So, while your new service is in the process of starting up, shippable has already begun draining the green instances of traffic. This can cause two problems:

1. If everything goes well, you have downtime during deployment, because your blue cluster is not ready to receive traffic, and your green cluster is already being brought down
2. Things may not go well, and your blue cluster may not ever reach healthy, because of a start up issue. Now your green and your blue are both down, and you're really red.

Shippable introduced a `stabilityDuration`, set in seconds, to address this. This is the amount of seconds it will watch that the ECS service has the prescribed number of containers before marking it successful, and bringing down the other cluster. There's more documentation on shippable's site [here](http://docs.shippable.com/deploy/deployment-method-blue-green/#validating-the-health-of-an-blue-green-deployment). 

This way, it will not bring down the old service, until the new service is marked as good.

---

_History for this post can be found at https://github.com/Ronnie76er/ronalleva-hugo/blob/master/content/posts/2018-07-02-health-check-grace-period-and-stability-duration-in-shippable.md. Please feel free to offer feedback or PRs on this post!_


