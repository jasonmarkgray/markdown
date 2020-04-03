---
HTTP endpoints for Webhooks
---

HTTP endpoints for webhooks allow you to listen to callback events from external sources such as Jira or GitHub.  Test

Specialized [event sources](/xl-release/how-to/webhook-event-source.html) are used to process events received by HTTP endpoints for webhooks. They can conditionally filter the incoming events and extract information into more specialized events, which can then be used in conjunction with [webhook event triggers](/xl-release/how-to/webhook-event-trigger.html) to perform actions such as creating a release, and map properties from the more specialized incoming event into template variables.

HTTP endpoints for webhooks can use two methods:
* POST endpoints is the most common, and reacts to POST HTTP events. POST endpoints can only consume JSON payloads.
* GET endpoints reacts to GET HTTP requests. GET requests have by definition no content, and usually send information through URL parameters instead.

When setting up an HTTP endpoint for webhooks, the user is required to provide at least four pieces of information:
* **HTTP method**: GET or POST
* **title**: the name of the endpoint
* **path**: the last segment of the URL this endpoint should be accessible at. This path must be globally unique across the XL Release system.
* **authentication method**: the method to verify the identity of the sender. Depending which method is selected, more information might be required - a secret key, for example.

> **Note:** HTTP endpoints for Webhooks and Event sources require users to have the [edit configuration](/xl-release/how-to/configure-release-teams-and-permissions/#configurations-permissions) permission.

## Webhook URLs

The URL of an XL Release HTTP endpoint for Webhooks will have the following form:
`XL_RELEASE_URL/webhooks/${path}`. For example, `http://localhost:5516/webhooks/webhook1`. If the version of XL Release has a non-root context such as http://localhost:5516/production-xlr/, then the endpoint will be http://localhost:5516/production-xlr/webhooks/webhook1.

> **Note**: a localhost instance of XL Release will not be accessible from anywhere but the same machine where XL Release is running.

When configuring HTTP endpoints for webhooks, you also need to expose XL Release to the external system's network. This often means having XL Release, or some specific endpoints, reachable from the public internet. There are various possible ways to expose your XL Release system to an external system, and which approach you choose will depend on your infrastructure.

## Webhook security

When configuring an HTTP endpoint for webhooks, it is required to select which type of authentication method to use to verify the identify of the sender and, in some cases - i.e. GitHub - also the integrity of the HTTP request's content.

Since opening an HTTP endpoint for webhooks in XL Release will expose the system to external HTTP requests, it is important to secure your connections. For example, if you use a publicly exposed endpoint, you should definitely avoid using the *No authentication* method.

The authentication method you use will depend entirely on which methods are supported by the external source that sends the requests.

Out of the box, and other than *No authentication*, only the GitHub authentication method is provided.

It is possible, but not recommended, to use the *Scripted Authentication (Jython)* authentication method and write a Jython script that will decide if an incoming request is considered authenticated or not. A Jython script will be considerably slower than a JVM-based solution and should not be used to authenticate HTTP requests that come at a frequent rate.

> **Note**: At present it is not possible for customers to write a JVM-based solution. The required interfaces will be published in a future release.

It is also possible to package a Jython script authentication method by extending the `events.CustomJythonAuthentication` type. This has the advantage of being much more user-friendly as it is possible to add custom properties to be used as configuration parameters for the script. It still has the limitation of being very slow compared to a JVM solution.

For more information, see [webhook plugins](/xl-release/webhooks/webhook-plugins.html).

### Scripted Authentication (Jython)

This script should set the variable `authenticated` to either `True` or `False`.

It can access the properties of the HTTP requests by using the following variables:

* `endpoint`: the HTTP endpoint for webhooks that recived the HttpRequestEvent
* `endpoint.title`: the name of this HTTP endpoint for webhooks
* `endpoint.path`: the path of this HTTP endpoint for webhooks
* `endpoint.method`: the HTTP method of this HTTP endpoint for webhooks. Can be either *POST* or *GET*.
* `endpoint.authentication`: the instance of the authentication for this endpoint.
* `config`: alias for `endpoint.authentication`, which contains the properties of this endpoint's authentication method configuration.
* `headers`: the headers of the current HTTP request this script is authenticating. Python dictionary.
* `params`: the URL parameters of the current HTTP request this script is authenticating. Python dictionary. Values are arrays of strings.
* `content`: the request body of the current HTTP request. A string. Only available if `endpoint.method` is *POST*.

#### Simple example:

```python
global params

def get_tokens():
    return params['token'] if 'token' in params else []

# True iff the 'token' parameter exists and one of its values matches 's3cr3t'
authenticated = 's3cr3t' in get_tokens()
```

## Create an HTTP endpoint for Webhooks


To define an HTTP endpoint for Webhooks:

1. In **Settings** > **Shared configuration** > **Webhooks and Events**, click *HTTP Endpoint For Webhooks* **+**.
2. Select either GET or POST Endpoint

3. Enter a name for this endpoint in the `Title` field
4. Enter a path for the endpoint to listen to. This will have the form `XL_RELEASE_URL/webhooks/${path}`. This path must be unique across the XL Release system. An error will notify the user if the chosen path is already taken upon saving.
5. Select an authentication method from the drop-down. For more information, see [Webhook security](#webhook-security):
    * No authentication
    * Github Authentication
    * Scripted Authentication (Jython)
6. Enable the event (default)
7. Click **Save**.

You can add more webhooks as needed. They can also be added per-folder in the folder **Configurations**, provided their path is globally unique.

Once the webhook is configured, you can use it when setting up webhook event triggers to create a release from a template. For more information, see [Webhook event triggers](/xl-release/how-to/webhook-event-trigger.html).

## HTTP endpoint for Webhooks and webhook event trigger tutorial

The tutorial below will guide you through the steps required to set up a very basic webhook with a webhook event trigger to start a release using a Curl request. For more advanced scenarios using event sources, see [webhook event sources](/xl-release/how-to/webhook-event-source.html).

This scenario will create a basic release from a Curl request, using a webhook and a webhook event trigger. You can improve this after testing by using a [webhook event source](/xl-release/how-to/webhook-event-source.html).

1. Set up a [basic HTTP endpoint for Webhooks](#create-an-http-endpoint-for-webhooks):
    * Select the "POST endpoint" option
    * Enter a name for this endpoint in the title field, i.e. `my endpoint`
    * Enter a path such as `unique_path`
    * Choose "No authentication"
    * Save the configuration
2. Set up a basic [webhook event trigger](/xl-release/how-to/webhook-event-trigger.html#create-a-webhook-event-trigger):
    * Create a new or select an existing folder under **Design** > **Folders**
    * With the folder selected, navigate to the **Templates** tab if not already on it
    * Open the "Add template" button and select the "Create new template" option
    * Give a name to the Template, i.e. `my template`
    * Click the "Create" button. You will redirected to the Release Flow view
    * From the "Show" menu, select *Variables*
    * Click the "New variable" button and create a new variablenamed `content` and of type "Text"
    * Navigate back to the folder
    * Navigate to the **Triggers** tab
    * Click the **Add trigger** button in the top-right corner
    * Select the "Webhook event trigger" option for the "Trigger type" field
    * Name the trigger, i.e. `my trigger`
    * Select which source this trigger will receive events from. Select the previously created HTTP endpoint for Webhooks `my endpoint`
    * The "Event type" field appears
    * Choose `HttpRequestEvent` as the event type
    * Choose a name for the release created by this trigger, i.e. `webhook release`
    * Optionally, enter a list of tags
    * Choose the template you just created (`my template`)
    * The "Template variables" section will appear. Click  and choose *Request Body* for the `content` variable.
    * Click the **Save** button.
3. Use a curl command to create a release:

```bash
curl -v \
    -H 'Content-type: application/json' \
    -X POST http://localhost:5516/webhooks/unique_path \
    -d '{"mydata":"123"}'
```
> If you have a different URL for your XL Release instance, enter that instead of `localhost`.
> Use the endpoint path you created above instead of `unique_path`, if you entered a different value.

4. Navigate to the Releases screen. You should see a new release with the name you defined (`webhook release`), and a variable with value `mydata:123`.
