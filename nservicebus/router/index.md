---
title: NServiceBus Router
summary: How to connect parts of the system that use different transports 
component: Router
related:
- samples/azure/azure-service-bus-msmq-bridge
- samples/msmq/sql-bridge
reviewed: 2018-09-05
---

`NServiceBus.Router` is a universal component that connects parts of an NServiceBus-based solution that otherwise could not talk to each other (e.g. because they use different transports or transport settings).

Unlike the [gateway](/nservicebus/gateway/) or the [wormhole](/nservicebus/wormhole/), the router handles both sending and publishing. Unlike the [bridge](/nservicebus/bridge/), the router can use site-based addressing to route messages between logically significant sites (just like the gateway does).

The router is transparent to the publishing and replying endpoint. That is:

 * The endpoint that *replies* to a message does not have to know if the initiating message came through a router. The reply will be routed automatically to the correct router and then forwarded to the initiating endpoint.
 * The endpoint that *publishes* events does not have to know if the subscribers are behind the router.


## Connecting to the router

Regular endpoints connect to the router using *connectors* that allow them to configure the routing

snippet: connector

The snippet above tells the endpoint that a designated router listens on queue `MyRouter` and that messages of type `MyMessage` should be sent to the endpoint `Receiver` via the router. It also tells the subscription infrastructure that the event `MyEvent` is published by the endpoint `Publisher` that is hosted behind the router.


## Router configuration

NServiceBus.Router is packaged as a host-agnostic library. It can be hosted e.g. inside a console application or a Windows service. It can also be co-hosted with regular NServiceBus endpoints in the same process.

The following snippet shows a simple MSMQ-to-RabbitMQ router configuration

snippet: two-way-router

The router has a simple life cycle:

snippet: lifecycle

The router can be configured to create all required queues on startup:

snippet: queue-creation


## Error handling

The router has a built-in retry strategy for error handling. It retries forwarding each message a number of times (*immediate retries*) and then moves it to the back of the input queue incrementing a *delayed retry* counter. If that counter reaches the maximum configured value, the message is moved to the poison message queue. The following snippet shows how it can be configured:

snippet: recoverability

In addition to immediate and delayed retries, the router has built-in outage detection through a *circuit breaker*. After a number of consecutive failures, the circuit breaker is triggered which causes the interface to enter the *throttled mode*. In this mode, the interface processes a single message at a time and pauses after each processing attempt. The interface goes back to the normal mode after the first successful processing attempt. When in the throttled mode, the router does not increment the delayed retries counter to prevent messages being sent to the poison message queue due to infrastructure outages.

## Topologies

A router consists of multiple [NServiceBus.Raw](/nservicebus/rawmessaging/) endpoints, called *interfaces*, and a *routing protocol* that controls how messages should be forwarded between them.


### Bridge

![Bridge](bridge.svg)

The arrows show the path of messages sent from `Endpoint A` to `Endpoint C` and from `Endpoint D` to `Endpoint B`. Each message is initially sent to the router queue and then forwarded to the destination queue. There is one additional *hop* compared to a direct communication between endpoints. The following snippet configures the built-in *static routing protocol* to forward messages between the router's interfaces.

snippet: simple-routing


### Multi-way routing

![Multi-way](multi-way.svg)

The router is not limited to only two interfaces but in case there are more than two interfaces, the routing protocol rules will be more complex and specific. The following snippet configures the built-in *static routing protocol* to forward messages to interfaces based on the prefix of the destination endpoint's name.

snippet: three-way-router

NOTE: All three interfaces use the same transport type (SQL Server Transport) but may use different settings, e.g. different database instances. This way, each part of the system (Sales, Shipping and Billing) can be autonomous and own its database server yet they can still exchange messages in the same way as if they were connected to a single shared instance.


### Backplane

![Backplane](backplane.svg)

Two or more routers can be connected together to form a _backplane_ topology. This setup often makes sense for geo-distributed systems. The following snippet configures the router hosted in the European part of the globally distributed system to route messages coming from outside via the Azure Storage Queues interface directly to the local endpoints and to route messages sent by local endpoints to either east or west United States through *designated gateway* routers.

NOTE: The *designated gateway* concept is not related to the `NServiceBus.Gateway` package. When the *designated gateway* is specified in the route, the message is forwarded to it instead of the actual destination.

snippet: backplane

NOTE: As an example the routing rules here use the `Site` property that can be set through the `SendOptions` object when sending messages. The backplane topology does not require site-based routing and can be configured e.g. using endpoint-based convention like in the multi-way routing example.
