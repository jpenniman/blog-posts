---
layout: post
title: Broadcasting Messages with Kafka
date: 2025-09-03
image: /blog/images/kafka-broadcast.jpeg
author: Jason M Penniman
excerpt: Domain events are being published to Kafka. The UI needs to subscribe to those events.
  How do we get every instance of the BFF to receive the message?
tags:
- Kafka
- Event Driven Architecture
---
## Problem

We're using Kafka for publishing domain business events. The UI needs to react to those events, so the BFF consumes the 
events and sends them to the UI over a GraphQL subscription.

When the BFF scales out, however, the replicas become part of the same consumer group, so not every replica consumes messages.

How do we get every message delivered to every BFF?

## Options

- Make every replica its own consumer group.
- Single partition with partition assignment and no consumer groups.
- Use an AMQP broker like RabbitMQ for these domain events.

But none of those were viable for us due to various business and security constaints. So now what?

## Solution

The Inbox Pattern! Well, a variant of it anyway.

We decided to use an inbox for processing theses events. The Kafka consumer in the BFF would consume the messages from the Kafka topic and save them into a table in the database--the inbox. Each BFF would process each message inbox.

Our implementation was very naive. Because we are just triggering UI refreshes, order and guaranteed delivery were not a concern.

