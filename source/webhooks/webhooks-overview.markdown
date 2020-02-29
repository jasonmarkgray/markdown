---
title: Webhooks overview
---

Webhooks, introduced in XL Release 9.6, are able to trigger releases as a reaction to specific HTTP requests made by an external source. This can potentially reduce the load of XL Release, while also reducing the pressure on external systems caused by polling methods where release triggers had to actively make requests to the third party system.

The core components of XL Release webhooks are:

* [HTTP endpoint for Webhooks](/xl-release/webhooks/http-endpoint-for-webhooks.html)
* [Webhook events](#events)
* [Event sources](/xl-release/webhooks/webhook-event-source.html)
* [Webhook event triggers](/xl-release/webhooks/webhook-event-triggers.html)
* [Messaging system (JMS)](#webhook-messaging-system)

HTTP endpoints for Webhooks are event sources that listen to HTTP requests and publish events of type `events.HttpRequestEvent`.

Events encapsulate the data flowing from HTTP endpoints for webhooks, through event sources and into webhook event triggers.

Event sources are used to process events from an event source or HTTP endpoint for webhooks. They can filter incoming events and potentially transform them into events of a different type.

Webhook event triggers are event consumers. Like an event source they subscribe to an event source or directly to an HTTP endpoint for webhooks, but instead of publishing another event, they create a release from a Template, like other trigger types.

HTTP endpoints for webhooks and other event sources publish events to the messaging system's queue. Events in the messaging system's queue are picked up and dispatched to event sources and webhook event triggers subscribed to the event source that published that event.

## HTTP endpoint for webhooks

HTTP endpoints for webhooks are *Shared configuration* items used to open a specific HTTP endpoint on XL Release.

An HTTP endpoint for wbhooks can be listening to *either* GET requests with no content, *or* POST requests with a JSON payload.

HTTP endpoints for webhooks are **not** protected by the regular system authentication, but they can be protected with custom authentication methods.

The external system that is configured to send webhook events to XL Release must be able to reach XL Release.

There are various possible ways to expose your XL Release system to an external system, and which approach you choose will depend on your infrastructure.

For more information, see [HTTP endpoint for Webhooks](/xl-release/how-to/http-endpoint-for-webhooks.html).

## Events

Events are the data structures of the information that flows from an event source to another and ultimately into a webhook event trigger.

Custom event types must be defined in a `synthetic.xml` file and must extend the `events.Event` type. For example, these are the definitions of the `events.Event` and `events.HttpRequestEvent` types:

```xml
<synthetic xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns="http://www.xebialabs.com/deployit/synthetic"
           xsi:schemaLocation="http://www.xebialabs.com/deployit/synthetic synthetic.xsd">

    <type type="events.Event" extends="udm.BaseConfigurationItem">
        <property name="sourceId" kind="string" required="true" label="Event Source" />
        <property name="received" kind="date" required="true" label="Received Timestamp" />
    </type>

    <type type="events.HttpRequestEvent" extends="events.Event">
        <property name="source" kind="ci" referenced-type="events.BaseSource" required="true" />
        <property name="headers" kind="map_of_string" label="Request Headers" />
        <property name="parameters" kind="map_of_string" label="Request Parameters" />
        <property name="content" kind="string" label="Request Body" />
    </type>

</synthetic>
```

* the `sourceId` property is the ID of the event source that published this event. This is true for all events, even specialized ones.
* the `received` property holds the timestamp of when the event was published by an event source or an HTTP endpoint for webhooks.
* the `source` property points to the HTTP endpoint for webhook that received the event from the external source.
* the `headers` property is a copy of the HTTP headers of the HTTP request received by the HTTP endpoint for webhooks.
* the `parameters` property is a copy of the HTTP parameters of the HTTP request received by the HTTP endpoint for webhooks.
* the `content` property is a copy of the HTTP request body. "GET endpoint" will leave this property empty, since GET requests have no content.

For more information about creating custom events, and customization in general, see the [webhooks plugin](/xl-release/how-to/webhook-plugins.html) tutorial.

## Event sources

Event sources, like HTTP endpoints for webhooks, are *Shared configuration* items and are used to process events of a certain type into events of another type.
Typically event sources are used to parse a generic `events.HttpRequestEvent` from an HTTP endpoint for Webhooks source into a specialized event type tailored to a specific 3rd party system.
Some event sources can also be configured to conditionally ignore events based on the content of the incoming event.

XL Release provides event sources for two Github webhook events: Pull Request and Push events, and one Scriptable Event Source (Jython).

For more information, see [Webhook event sources](/xl-release/how-to/webhook-event-source.html).

## Webhook event triggers

Webhook event triggers are a class of release triggers that execute upon receiving an event from the event source to which they are subscribed.

When configuring a webhook event trigger, the user can define, for each required template variable, which value to use. It can be a manually entered value, a value from a folder variable, a value from a global variable, or the value of a property of the received event.

Refer to the full article regarding [Webhook event Triggers](/xl-release/how-to/webhook-event-trigger.html) for more information.

## Webhook messaging system

Event sources and webhook event triggers receive and publish events using a JMS queue.

XL Release 9.6 ships by default with an embedded JMS messaging queue system. However, if you wish to use your own message system, you can configure the `xl-release.conf` file with the following options:
```json
xl.queue {
        embedded = true
        # the following properties are only used when embedded = false
        url = "amqp://jms.my-intranet:61616"
        username = "myUsername"
        password = "myPassword"
        queueName = "myQueueName"
    }
```

If you want to use the webhooks feature in a High Availability (cluster mode) setup, the JMS queue **cannot** be embedded. It must be external and shared by all nodes in the XL Release cluster.

In cluster mode, `xl.queue.embedded` must be set to `false` and the rest of the `xl.queue` configuration section must be properly configured to point to an existing JMS queue.
