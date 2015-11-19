---
layout: post
title: "Microservices at Wrapp"
date:   2015-10-09
tags: [Microservices, Docker]
---

This post is based on a talk by [Eskil Andréen](https://se.linkedin.com/in/eskila), CTO of Wrapp. Some parts are directly taken from his presentation, while other are my personal view on the topic. If you like any of what you read thank Eskil if you don't blame me :)


## About Wrapp

[Wrapp](http://www.wrapp.com/) was founded around 4 years ago, the first version of the product was an application to send digital gift cards to your friends. A bit more than a year ago the company pivoted, being the new goal to build a loyalty platform that integrates with your credit/debit card and automatically gives discounts based on your purchase behavior.

When it was decided to pivot and close the old service, we took the decision of getting rid of all our backend infrastructure and start from scratch, with a new architecture based on microservices. The main requirements for the new architecture were scalability, availability and ease to introduce changes.

We are currently 14 people in the tech team, and all our infrastructure runs on Amazon Web Services (AWS).


## Why microservices at Wrapp?

At the time when the pivot was started our backend consisted of a few monolithic applications written in Python, that after three years of fast paced  development were messy and full of technical debt. To make things worse, there were a lot of shared dependencies between the applications. This required a complex infrastructure in order to manage the inter dependencies between the applications. We also had a tone of complex tests, that made introducing changes to the existing code harder.

We hoped that microservices would help to prevent those issues. With microservices each component is small and simple, which means that it is more feasible to keep the code well organized and less tests are required. All this makes modifying and adding new functionality faster.

We also wanted to make work more rewarding by allowing developers to design their components as they wish, with the tools they want (language, libraries, frameworks...) and because smaller units of work are more satisfying to complete. Finally, encouraging collective ownership by involving everyone in the design and implementation of complete parts of the system.


## How does Wrapp look now

One year after starting the complete rewrite of our backend, we have launched a new product and doubled the size of the tech team. Our backend is formed by around 50 services, each one run as Docker container, mainly written in Python and Go; Everything that is important for our product is a service. In this word cloud you can see most of the services that conform Wrapp's backend:

<figure>
	<img src="/images/microservices-cloud.png" alt="Microservice cloud"></a>
</figure>


### Tenancy, or where to run each service?

We started with the simplest approach, **single-tenancy**: Each server runs a single type of service. The main benefit of this approach is its simplicity, the problem though is that it is wasteful as many services are idling most of the time or are very small and don't use all the resources of the machine where they run.

To solve that we evolved to **static multi-tenancy**: Each server runs a predefined set of services. This partially solves the issues of single-tenancy, but forces you to manually decide how to group services.

Finally we are using **dynamic multi-tenancy**: Services are dynamically assigned to hosts without any per-host static configuration. We use [Amazon ECS](https://aws.amazon.com/ecs/) to manage the placement of our services.

<figure>
	<img src="/images/multitenancy.png" alt="Where to run each service?"></a>
	<figcaption>Where to run each service?</figcaption>
</figure>


### Service discovery

After a few iterations we use a solution based on DNS, [Consul](https://www.consul.io/), [Registrator](http://gliderlabs.com/registrator) and [HAProxy](http://www.haproxy.org/) that transparently handles addition and deletion of services.

We assign a different private IP to every service and for convenience we map service names to their private IP using DNS.

Registrator, Consul Agent and HAProxy run in each server of the cluster:

* Registrator inspects Docker containers as they come online/offline and informs the local Consul Agent of the services that they run.
* Consul Agent periodically registers the services that are running in the machine to the Consul Catalog and refreshes the configuration of the local HAProxy to include all the services listed in the Catalog via consul-template.
* HAProxy is configured to listen for internal requests to the private IPs assigned to the services running in the cluster and load balances the requests among all the available instances of each service.

To better understand this mechanism, lets use an example; Suppose we are adding a new service into our cluster, the users service, this is what happens:

* We assign the private IP ```192.186.10.1``` to the users service.
* We add a DNS name ```users.internal.ourdomain``` mapping to the private IP of the users service.
* The new service is deployed, and it starts running in a subset of our servers.
* Registrator running in those servers will detect a new Docker container running the users service and will inform the Consul Agent that will register it to the Consul Catalog.
* Eventually the Consul agent running in each server will update the local HAProxy configuration adding a new backend to serve requests to the users service IP ```192.186.10.1```.
* Finally when a service in the cluster issues a request to ```users.internal.ourdomain``` the DNS name will be resolved to the private IP and the local HAProxy will receive the request and forward it to one of the available instances of the users service.

<figure>
	<img src="/images/service-discovery.png" alt="Performing a request"></a>
</figure>

By default, all microservices are only accessible from within our network. For services that need to be accessible from the internet, we have a service connected to a public load balancer that takes care of receiving requests and forward them to the specified internal service.


### Inter-service Communication

Each service exposes a REST API and inter-service communication is done over HTTP with JSON payloads. Communication can be synchronous HTTP requests or it can be done using the eventbus.

The eventbus is in itself a microservice that offers publish/subscribe functionality. It is built on top of Amazon [SQS](https://aws.amazon.com/sqs/) and [SNS](https://aws.amazon.com/sns/). It guarantees at least once delivery and automatically retries failed requests. We use this way of communication for all the tasks that don't require an immediate response.

Given the amount of services and requests involved in performing any complex task, it is not strange that some request will fail when performing a task. For this reason is a good practice to have retrials on requests. This means that requests must be idempotent, issuing the same request multiple times should always produce the same result. From our experience, having retrials and idempotent requests really helps in dealing with the complexity that aries from working in a distributed system.


### Monitoring

It is a complex system with lots of moving parts, so there are many things to monitor and we use a few different strategies:

* Monitor logs for warnings, errors and deviations.
* Monitor Network traffic: number of requests, latency, percentage or errors.
* Define and monitor KPIs for each service important service.

You can check out this blog post [Monitoring microservices with haproxy and riemann](/2015/07/12/monitoring-microservices-with-haproxy-and-riemann/) to know more about our monitoring tools.


## Conclusion

After one year working with microservices they have brought a lot of good things to our organization and have been overall a great choice, confirming most of our initial expectations. We have a relatively big and complex system, composed by many small and simple services. This means that it is simple to understand, extend and even replace individual components of our backend, giving us great flexibility. We also feel that microservices have a great impact in developer satisfaction.

We also face a few issues from using microservices. First of all it was a big upfront time investment to develop the required infrastructure to support microservices: deployment, service discovery, monitoring...

Secondly, distributed systems are hard and with microservices we face all the issues that come with them regarding consistency, atomicity and concurrency. It is also hard to cleanly split complex functionalities between multiple services and model the interactions between them while keeping the involved services decoupled.

Finally there is an important overhead in implementing and maintaining HTTP interfaces and clients for each service and we face higher latency due to multiple http hops. Any common task performed by the clients of our API requires tens of HTTP requests between different services.

A special mention is also reserved for testing; Our approach is having a strict contract for the interface of each component, and focus our testing efforts in internal service tests that make sure that the contract is satisfied. But it is hard to be sure that interactions between services will work as expected when they are put together and we haven't found a good way of performing automatic integration tests.

For all these reasons is probably not a good idea to use microservies for small projects or those that have well defined and non-changing requirements, as then the benefits of microservices are less obvious.

<figure>
	<img src="/images/need-microservices-flowchart.png" alt="Need microservices"></a>
	<figcaption>Shoud you use microservies? (Source: http://www.stavros.io/posts/microservices-cargo-cult)</figcaption>
</figure>
