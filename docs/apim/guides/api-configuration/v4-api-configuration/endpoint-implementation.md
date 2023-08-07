---
description: This article discusses the implementation details of v4 API endpoints
---

# Endpoint implementation

## Overview

Gravitee supports several different message brokers. This page describes the integrations Gravitee uses to enable Kafka, MQTT, RabbitMQ, and Solace endpoints for v4 API definitions. These rely of the following terminology and functionality:

* **Request-Id**: A Universally Unique Identifier (UUID) generated for any new request. This can be overridden using `X-Gravitee-Request-Id`as a Header or Query parameter.
* **Transaction-Id**: A UUID generated for any new request. This can be overridden using `X-Gravitee-Transaction-Id`as a Header or Query parameter.
* **Client-Identifier**: Inferred from the subscription attached to the request. It is either the subscription ID, or, with a Keyless plan, a hash of the remote address. The **Client-Identifier** can be provided by the client via the header `X-Gravitee-Client-Identifier`. In this case, the value used by Gravitee will be the original inferred value suffixed with the provided overridden value.

## Kafka

### Subscribe

For each incoming request, a consumer is created and will persist until the request terminates. The Kafka endpoint retrieves information from the request to create a dedicated consumer characterized by:

* **ConsumerGroup:** The consumer group is computed from the request's client identifier. Kafka doesn't offer a way to manually create a consumer group, it is done through a new consumer instance. This consumer group is used to load balance consumption. See the [Kafka documentation](https://docs.confluent.io/platform/current/clients/consumer.html#concepts) for more information.
* **ClientId:** A client ID is generated for the consumer with the format `gio-apim-consumer-<first part of uuid>`, e.g., `gio-apim-consumer-a0eebc99`.
* **AutoOffsetReset:** The `auto-offset-reset` of the API is managed at the endpoint level and cannot be overridden by request.

### Offset selection

By default, the consumer that is created will either resume from where it left off or use the `auto-offset-reset` configuration to position itself at the beginning or end of the topic.&#x20;

Offsets are determined by partitions, which results in numerous possible mappings. Due to the inherent complexity of offset selection, Gravitee has introduced a mechanism to target a specific position on a Kafka topic.&#x20;

Given a compatible entrypoint (SSE, HTTP GET) and by using at-most-once or at-least-once QoS, it is possible to specify a last event ID. Refer to the entrypoint documentation to know how to do so. The format is encoded by default but follows the pattern below:

```yaml
<topic1>@<partition11>#<offset11>,<partition12>#<offset12>;<topic2>@<partition21>#<offset21>,<partition22>#<offset22>...
```

### Publish

A shared producer is created by the endpoint and reused for all requests that have the same configuration. A shared producer is characterized by:

* **ClientId:** The client ID of the producer is generated with the format `gio-apim-producer-<first part of uuid>`, e.g., `gio-apim-producer-a0eebc99`.
* **Partitioning:** The only supported method for targeting a specific partition is to define a key and rely on the built-in partitioning mechanism. Kafka uses the key to compute the associated partition ( hash(key) % nm of partition). Repeated use of the same key on each message guarantees that messages are relegated to the same partition and order is maintained. Gravitee doesn't support overriding this mechanism to manually set the partition. To set a key on a message, the attribute `gravitee.attribute.kafka.recordKey` must be added to the message.
* **QoS:** The producer uses none, auto, at-least-once, or at-most-once QoS

## RabbitMQ

### Subscribe

#### Exchange

The endpoint will declare the exchange with the option provided by the configuration at the API level. The exchange name can be overridden with the attribute `rabbitmq.exchange`**.**

If the exchange options provided are incompatible with the existing exchange found on Rabbit, the request will be interrupted with an error.

#### Queue

A queue will be created using the client identifier of the request following this format: `gravitee/gio-gateway/<clientIdentifier>`**.**

The created queue will have different options depending on the QoS applied on the entrypoint:

* None:
  * durable = false
  * autoDelete = true
* Auto
  * durable = true
  * autoDelete = false
* **Other not supported**

If the queue already exists, the messages would be load-balanced between both clients.

#### RoutingKey

In order to route the right messages to the queue, A routing key is used from the API configuration and used to create the binding between the exchange and the queue.

The routing key can be overridden with the attribute `rabbitmq.routingKey.`

### QoS

* None

Strategy applying a high throughput, low latency, no durability, and no reliability.

> The broker forgets about a message as soon as it has sent it to the consumer. Use this mode if downstream subscribers are very fast, at least faster than the flow of inbound messages. Messages will pile up in the JVM process memory if subscribers are not able to cope with the flow of messages, leading to out-of-memory errors. Note this mode uses the auto-acknowledgment mode when registering the RabbitMQ Consumer.

* Auto

Strategy balancing between performances and quality.

When the entrypoint supports manual ack, the strategy will use it. Otherwise, it will use auto ack coming from the RabbitMQ Reactor library:

> _With this mode, messages are acknowledged right after their arrival, in the Flux#doOnNext callback. This can help to cope with the flow of messages, avoiding the downstream subscribers to be overwhelmed. Note this mode does not use the auto-acknowledgment mode when registering the RabbitMQ Consumer. In this case, consumeAutoAck means messages are automatically acknowledged by the library in one the Flux hooks._