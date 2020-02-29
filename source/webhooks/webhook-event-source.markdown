---
title: Webhook event sources
---

Webhook event sources allow you to transform incoming events into other types of events. For example the "Github Pull Request Event Source" is able to transform an incoming "HTTP Request Event" into a "Github Pull Request Event", provided the request body of the HTTP Request Event conforms with a Github Pull Request Event payload as defined by Github.

Optionally, an event source can conditionally ignore the incoming event. For example, the Github pull request event source can be configured to ignore events depending on the value of the "action" field, so that only events about pull requests being "opened" are processed.

* You can extend the `events.Event` type to define custom events.
* You can extend the `events.CustomJythonEventSource` type to create a custom event source. See [Webhooks plugins](/xl-release/how-to/webhook-plugins.html) for more information about writing custom event sources.

> **Note**: you must have an [HTTP endpoint for webhooks](/xl-release/how-to/http-endpoint-for-webhooks.html) already configured to create an event source.

## GitHub Event Sources

The `Github: Pull Request Event Source` allow you to receive GitHub webhook-based pull request events, while the `Github: Push Event Source` if for github push events. These event sources  transform `events.HttpRequestEvent`s into useful events for a [webhook event trigger](/xl-release/how-to/webhook-event-trigger.html): `github.PullRequestEvent` and `github.PushEvent` respectively. You can filter the data that comes in from GitHub, and pull a set of fields that can be used when populating the values in a webhook event trigger. This can be used to set values such as release name, release tags and other variables in a release.

### Github events types definitions

```xml
    <type type="github.PullRequestEvent" extends="events.Event">
        <property name="number" kind="integer"/>
        <property name="repository" kind="ci" referenced-type="github.Repository" nested="true"/>
        <property name="sender" kind="ci" referenced-type="github.User" nested="true"/>
        <property name="action" kind="string"/>
        <property name="pull_request" kind="ci" referenced-type="github.PullRequest" nested="true"/>
    </type>

    <type type="github.PushEvent" extends="events.Event">
        <property name="repository" kind="ci" referenced-type="github.Repository" nested="true"/>
        <property name="before" kind="string"/>
        <property name="pusher" kind="ci" referenced-type="github.UserEmail" nested="true"/>
        <property name="ref" kind="string"/>
        <property name="commits" kind="list_of_ci" referenced-type="github.Commit" as-containment="true"/>
        <property name="after" kind="string"/>
        <property name="forced" kind="boolean"/>
        <property name="sender" kind="ci" referenced-type="github.User" nested="true"/>
        <property name="head_commit" kind="ci" referenced-type="github.Commit" nested="true"/>
        <property name="compare" kind="string"/>
        <property name="deleted" kind="boolean"/>
        <property name="created" kind="boolean"/>
    </type>
```

The nested type definitions (like `github.User`) are omitted here, but you can use the `/metadata/type/{:typeName}` endpoint to retrieve such information. For example:

```bash
curl -u admin:admin -H 'Accept: application/json' http://localhost:5516/metadata/type/github.User
```

> **Note**: use your own login credentials instead of 'admin:admin' and the url of your XL Release instance instead of http://localhost:5516

GitHub events in XL Release are modelled directly from the GitHub specification:

* https://developer.github.com/v3/activity/events/types/#pullrequestevent
* https://developer.github.com/v3/activity/events/types/#pushevent

Properties named `id` or `name` in GitHub are prefixed with an underscore ('\_') in XL Release since those are reserved names.

## GitHub: Pull Request Event Source tutorial

This scenario will use the `GitHub: Pull Request Event Source` to trigger a release from a GitHub pull request event.

### Prerequisites

* A GitHub account and a repository to which you have admin privileges.
* A version of XL Release that is publicly accessible from GitHub
* A folder and a release template (in that folder) you wish to test on. A basic Manual task will be enough to start the release.
* For more advanced testing, you could set up template variables to hold GitHub Pull Request Event properties, and GitHub-related tasks to test a complete workflow. The template variables should match the GitHub field type, for example a *Number* variable to hold an integer property, and a *Date* variable to hold a date.

### Step-by-step procedure

1. Set up an [HTTP endpoint for Webhooks](/xl-release/how-to/Bt-for-webhooks.html#create-an-http-endpoint-or-webhooks):
    * Select the "POST endpoint" option
    * Enter a name for this endpoint in the title field, i.e. `github: my-repo endpoint`
    * Enter a path such as `github-my-repo`
    * Choose "Github Authentication"
    * Enter a secret key, i.e. `1234567890`
    * Save the configuration
2. Add the webhook in GitHub:
    * In your repository, navigate to **Settings** > **Webhooks**
    * In *Payload URL*, enter the webhook address in the form `XL_RELEASE_URL/webhooks/${path}`, where path is the path chosen when creating the HTTP endpoint for Webhooks, `github-my-repo` in this example.
    * In *Content type*, select `application/json`
    * Enter the secret, use the same value used when setting up the HTTP endpoint for Webhooks, `1234567890` in this example.
    * For the events to trigger the webhook, select *Let me select individual events*, then make sure *Pull requests* are selected
    * Click **Update webhook**
    * Test the webhook below. You should get a 200 success response.
3. Create a [GitHub Pull Request Event Source](#github-event-sources):
    * In XL Release, navigate to **Settings** > **Shared configuration** > **Webhooks and Events** > **Github: Pull Request Event Source** > Add new
    * Add a title, i.e. `github: my-repo pull requests`
    * In *Source*, select the endpoint you created above, `github: my-repo endpoint` in this example
    * Make sure the `enabled` checkbox is enabled.
    * For now you do not need to enable any of the filtering fields
    * Click **Save**



4. In your test folder, set up a [webhook event trigger](/xl-release/how-to/webhook-event-trigger.html#create-a-webhook-event-trigger):
    * Navigate to **Folders** > [Folder name] > **Triggers** > **Add Trigger**, and select *Webhook event trigger*
    * Give it a title, i.e. `github: my-repo pull request trigger`
    * In *Event Source*, select the GitHub event parser you created above, i.e. `github: my-repo pull requests`
    * In *Event Type*, select *github.PullRequestEvent*
    * In *Release title*, click  and select *Title - pull_request.title*. You can enter search text to narrow down the available fields
    * In *Template*, select the test release template
    * If you had created variables for mapping, you could set up mapping in the *Template variables* section that opens, using the same procedure for setting the *Release title* above
    * Click **Save**
5. Trigger a release:
    * In your GitHub repository, create a new pull request
    * In XL Release, you should see that a new release was created with the name of the pull request.


## Scripted event source (Jython)

The Scripted Event Source (Jython) can be used to quickly define new event sources. You can define their behavior by writing Jython scripts in the fields provided.

1. In **Settings** > **Shared configuration**, create a new **Scripted Event Source (Jython)**
1. Enter a title
1. Select the event source. This can be an HTTP endpoint for Webhooks or another event source.
1. Select the *Input Event Type* from the list. This list will depend on which event source was selected in the previous step.
1. Select the *Output Event Type* from the list. This will determine the type of event that will be created from the processor. To define your own event types, you need to declare them in your `synthetic.xml` file. It should extend the `events.Event` type.
1. Enter a *Filter Script* and a *Map Script*. See below for more details.

### Scripted event source (Jython) scripts

Two types of scripts can be used to transform an incoming event into the output event:

* **Filter Script** - evaluates the message for acceptance criteria and either passes it on or ignore it.
* **Map Script** - script to transform the input event into the output event.

In both scripts you can access the incoming event using the `input` variable. It's properties can be accessed using the dot notation, i.e. if your event has a property named `content`, you can access it with `input.content`. It also works for nested properties - properties whose kind is `ci` and which have the `nested="true"` attribute.

A helper function called `CI` can be used to create the output event in the **Map Script**: it takes two arguments, the first is the name of the configuration item's type - for example `test.MyEvent` - and the second is a dictionary containing the values to assign each property.

Refer to the following tutorial for a more concrete example.

### Scripted event source (Jython) tutorial

1. Add the following to your `synthetic.xml`, for example `XL_RELEASE_HOME/ext/synthetic.xml`:

```xml
<type type="test.MyInnerObject" extends="udm.BaseConfigurationItem">
    <property name="myinnervalue" kind="integer"/>
</type>

<type type="test.MyEvent" extends="events.Event" virtual="false" label="My Event">
    <property name="myvalue" kind="integer"/>
    <property name="mymessage" kind="string"/>
    <property name="myobject" kind="ci" referenced-type="test.MyInnerObject" nested="true"/>
</type>
```
2. Restart XL Release
3. Create a [POST endpoint](/xl-release/how-to/http-endpoint-for-webhooks) without authentication, use `test` for its **path**.
4. In **Settings** > **Shared configurations**, create a new *Scripted Event Source (Jython)*:
    * Give it a name, for example `my scripted event source`
    * Connect it to the endpoint you created above
    * In *Input Event Type*, select `HttpRequestEvent`
    * In *Output Event Type*, select `My Event`
5. Use the following scripts:

**Filter script**
```python
global input

accepted = 'myvalue' in input.parameters and int(input.parameters['myvalue']) > 10
```

**Map script**

```python
import json

global input
global CI

data = json.loads(input.content)
output = CI('test.MyEvent', {
    'myvalue': int(input.parameters['myvalue']) if 'myvalue' in input.parameters else -1,
    'mymessage': data['mymessage'] if 'mymessage' in data else None,
    'myobject': CI('test.MyInnerObject', {
        'myinnervalue': data['inner']['value'] if ('inner' in data) and ('value' in data['inner']) else None
     })
})
```

6. In **Design** > **Folder**, create a new folder, and create a new release template.
    * In the template add a variable with an integer variable *myvalue*
7. In the **Triggers** of the same folder, create a new [webhook event trigger](/xl-release/how-to/webhook-event-trigger.html#create-a-webhook-event-trigger)
    * Connect it to the processor you created above, i.e. `my scripted event source`
    * Add a title
    * In *Input event type*, use `My Event`
    * Choose the template you created above
    * In the variable section, clickand choose *myvalue* from the list
    * Save the trigger

#### Test the event source

##### Test the filter script:

From a terminal:

```bash
curl \
    -H 'Content-type: application/json' \
    -X POST \
    'http://localhost:5516/webhooks/test?myvalue=4' \
    -d '{"mymessage": "Hello", "inner": {"value": 42}}'
```
> Note: use the actual URL of your XL Release instance instead of http://localhost:5516.

You will see that the release is not created because the filter only accepts values over 10.


###### Test the map script:

From a terminal:

```bash
curl \
    -H 'Content-type: application/json' \
    -X POST \
    'http://localhost:5516/webhooks/test?myvalue=12' \
    -d '{"mymessage": "Hello", "inner": {"value": 42}}'
```

> Note: use the actual URL of your XL Release instance instead of http://localhost:5516.

You will see that the new release was created with a `myvalue` variable = 12.


#### Package a custom event source as a plugin

For more information, see [Webhooks plugins](/xl-release/how-to/webhook-plugins.html).
