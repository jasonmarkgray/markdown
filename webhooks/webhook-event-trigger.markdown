---
title: Webhook event triggers
product:
- xl-release
category:
- Release components
subject:
- Webhooks
tags:
- webhooks
- time
- schedule
- trigger
order: 400
---

Webhook event triggers listen for events from event sources such as [webhooks endpoints](/xl-release/how-to/webhook-endpoints/) or [webhook event sources](/xl-release/how-to/webhook-event-sources/) to create releases. Webhook event triggers:

1. Subscribe to an event source, defined in the **Webhooks and Events** section of the **Shared configuration** screen and the folder **Configuration** tab.
2. React to events published by that event source by creating releases.
3. Configure the create release action by populating the required template variables according to rules that you define. See [mapping properties](#mapping-properties) for more information.
4. Create releases as soon as the event arrives with the values populated in the mapping process.

For a basic tutorial, see [HTTP endpoint for Webhooks and webhook event trigger tutorial](/xl-release/how-to/http-endpoint-for-webhooks.html#http-endpoint-for-webhooks-and-webhook-event-trigger-tutorial).

## Create a webhook event trigger

Once you have already defined a webhook [event source](/xl-release/how-to/webhook-event-sources/), you can create a webhook event trigger.

1. Under **Design** > **Folders**, create a new folder
1. With the newly created folder selected, create a new Template
1. add some variables to the template
1. Navigate to **Triggers** tab of the folder (Note: you can also create a trigger from inside a template: Using the **Show** menu, select "Triggers". After clicking "Add trigger", the trigger form will be shown with the template preselected, other than that, all the other steps are the same)
1. Click the "Add trigger" button
1. Select the "Webhook event trigger" option for the **Trigger type** field
1. Give it a title and a description, and ensure it is enabled
1. Select the **Event source** this trigger should react to. The list will include all the event sources defined in this folder and its parents
1. Once the **Event source** is selected, select the **Event type**, in most case you should leave the default value, which is defined by the Event source's output event type.
1. Select the previously created template
1. In the **Release title**, you can either enter a title manually or click ![field selector](../images/multiselect-icon.png) to select a source from the event data to populate the title.
1. Select a template from the list to create the release from
    * If the template contains variables, a new **Template variables** section will open. See below for more details.
1. In **Tags**, either enter a list of new tags separated by pressing *Enter*, or click![field selector](../images/multiselect-icon.png) to select a source to populate tags
1. If there are template variables, the fields will already be populated with the default values. You can either overwrite this, or use the ![field selector](../images/multiselect-icon.png) icon to select a source to populate the variables.
1. Click **Save**.

## Mapping properties

Field properties in webhook event triggers can either be entered manually, or copied from a field in the event. For example, the event source values can be used to populate the Release title. Click the field selector icon ![field selector](../images/multiselect-icon.png) to use event source values.

![Using field selector](../images/using-field-selector.png)

## Webhook event permissions

Webhook event triggers have the same permissions as other trigger types. Users should have "Manage triggers" permissions to create or edit webhook event triggers.
