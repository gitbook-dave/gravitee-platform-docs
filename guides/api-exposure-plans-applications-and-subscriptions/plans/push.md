---
description: This page describes the Push authentication type
---

# Push

## Introduction

A Push plan is used when an API contains an entrypoint that sends message payloads to API consumers (e.g., Webhook). This type of plan is unique in that the security configuration is defined by the API consumer, in the subscription request created in the Developer Portal. For example, when subscribing to a Webhook entrypoint, the API consumer specifies the target URL and authentication for the Gateway to use when sending messages.

{% hint style="info" %}
Push plans do not apply to SSE entrypoints. Although messages are pushed from the server, the client application initiates message consumption.
{% endhint %}

## Configuration

Push plans have the same configuration options as [Keyless plans](keyless.md) in APIM. The bulk of the configuration for a Push plan is set by the API consumer in the Developer Portal, and the content of the configuration varies by entrypoint type.

{% hint style="info" %}
Gravitee currently supports Push plans for Webhook entrypoints
{% endhint %}