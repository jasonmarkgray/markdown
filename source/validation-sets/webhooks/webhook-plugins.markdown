---
title: Webhooks plugins tutorial
product:
- xl-release
category:
- Release components
subject:
- Webhooks
tags:
- webhooks
- events
order: 500
---

In this tutorial we are going to create a custom authentication method for [HTTP endpoint for webhooks](/xl-release/webhooks/http-endpoint-for-webhooks.html), a custom event type, and a custom event source.

The tutorial plugin will be called `xlr-mywebhooks-plugin`.

The authentication method will be able to authenticate HTTP requests based on an URL parameter's value. The user must configure the parameter name and a secret token.

The custom event type will model a very simple chat message with a user, a topic and the message itself. The event source will be able to read such message from both GET and POST requests, and will have some basic filtering capabilities.

## Plugin structure

1. First create a directory for the plugin:

`mkdir xlr-mywebhooks-plugin`

2. Create the structure:

```bash
touch synthetic.xml
mkdir mywebhooks/
touch mywebhooks/UrlParameterTokenAuthentication.py
touch mywebhooks/ChatMessageEventSource.py
```

The `synthetic.xml` file will contain the custom types definitions while the `mywebhooks` directory will contain the python scripts.

3. Enter the following text into the `synthetic.xml` file:

```xml
<synthetic xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns="http://www.xebialabs.com/deployit/synthetic"
           xsi:schemaLocation="http://www.xebialabs.com/deployit/synthetic synthetic.xsd">

</synthetic>
```

That is what an empty `synthetic.xml` file looks like, and you will add type definitions inside the `<synthetic>` node.

## Custom authentication method: URL Parameter Token

1. Open the `synthetic.xml` file add add the type definition of our custom authentication method:

```xml
<synthetic xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns="http://www.xebialabs.com/deployit/synthetic"
           xsi:schemaLocation="http://www.xebialabs.com/deployit/synthetic synthetic.xsd">

    <type type="mywebhooks.UrlParameterTokenAuthentication"
        extends="events.CustomJythonAuthentication"
        label="URL Parameter Token"
        description="Verify that the value of a given URL parameter matches the secret token.">
        <property name="parameter" kind="string" required="true"
            label="URL parameter"
            description="URL parameter name containing the token."/>
        <property name="token" kind="string" password="true" required="true"
            label="Token"
            description="Secret token value."/>
    </type>

</synthetic>
```

2. Open the `mywebhooks/UrlParameterTokenAuthentication.py` file and write this script:

```python
# required to decrypt password properties. Can only be used by packaged scripts.
from com.xebialabs.deployit.util import PasswordEncrypter

# the 'events.WebhookEndpoint' instance.
global endpoint
# the `mywebhooks.UrlParameterTokenAuthentication` instance. Holds the `parameter` and `token` properties
global config
# the HTTP headers. Python dictionary
global headers
# the URL parameters. Python dictionary. Each value is an array of strings.
global params
# the HTTP request body. Only available when endpoint.method == "POST"
global payload

# let's get the PasswordEncrypter instance
pe = PasswordEncrypter.getInstance()


# find the URL parameter named `config.parameter` and return its values, or an empty array if not found
def getTokens():
    return params[config.parameter] if config.parameter in params else []

# True if one of the values of the `config.parameter` URL parameter is the decrypted `config.token`.
authenticated = pe.ensureDecrypted(config.token) in getTokens()
```


### Test the new custom authentication method

#### Setup a test endpoint

1. Add the type definition from `synthetic.xml` to `XL_RELEASE_HOME/ext/synthetic.xml`
1. Copy your `mywebhooks` directory to your `XL_RELEASE_HOME/ext/` directory
1. Restart XL Release
1. Under **Settings** > **Shared configuration** > **Webhooks and Event**, create a new *HTTP GET endpoint for Webhooks*
1. Name it, for example `My Test Endpoint (GET)`
1. Set the path, for example `test_endpoint_get`
1. Under authentication, a new option named 'URL Parameter Token" should be available
1. Select 'URL Parameter Token'
1. Configure the **parameter** field: type `token`
1. Configure the **token** field: type `tok3n`
1. Save the new HTTP endpoint for webhooks

#### Verify the test endpoint with the custom authentication method

1. Check that it rejects requests with the bad token value:

```bash
curl -v 'http://your-xl-release-url/webhooks/test_endpoint_get?token=wr0ng'
```
    > *   Trying ::1:5516...
    > * TCP_NODELAY set
    > * Connected to localhost (::1) port 5516 (#0)
    > > GET /webhooks/test_endpoint_get?token=wr0ng HTTP/1.1
    > > Host: localhost:5516
    > > User-Agent: curl/7.66.0
    > > Accept: */*
    > >
    > * Mark bundle as not supporting multiuse
    > < HTTP/1.1 401 Unauthorized
    > < Date: Mon, 24 Feb 2020 15:59:00 GMT
    > < X-XSS-Protection: 1; mode=block
    > < X-Content-Type-Options: nosniff
    > < Content-Type: application/json
    > < Content-Length: 238
    > <
    > * Connection #0 to host localhost left intact
    > Unauthorized request for 'Configuration/Custom/Configurationc2f85e3a3d424d45a81f4b87fe32bc5e (My Test Endpoint (GET))' (authentication method: com.xebialabs.xlrelease.webhooks.authentication.CustomJythonAuthenticationMethod)%  

1. Check that it rejects requests with the bad parameter name:

```bash
curl -v -X GET 'http://your-xl-release-url/webhooks/test_endpoint_get?wrong=tok3n'
```

    > *   Trying ::1:5516...
    > * TCP_NODELAY set
    > * Connected to localhost (::1) port 5516 (#0)
    > > GET /webhooks/test_endpoint_get?wrong=tok3n HTTP/1.1
    > > Host: localhost:5516
    > > User-Agent: curl/7.66.0
    > > Accept: */*
    > >
    > * Mark bundle as not supporting multiuse
    > < HTTP/1.1 401 Unauthorized
    > < Date: Mon, 24 Feb 2020 15:57:49 GMT
    > < X-XSS-Protection: 1; mode=block
    > < X-Content-Type-Options: nosniff
    > < Content-Type: application/json
    > < Content-Length: 238
    > <
    > * Connection #0 to host localhost left intact
    > Unauthorized request for 'Configuration/Custom/Configurationc2f85e3a3d424d45a81f4b87fe32bc5e (My Test Endpoint (GET))' (authentication method: com.xebialabs.xlrelease.webhooks.authentication.CustomJythonAuthenticationMethod)%

1. Check that it accepts requests with the correct parameter name and value:

```bash
curl -v -X GET 'http://your-xl-release-url/webhooks/test_endpoint_get?token=tok3n'
```

    > *   Trying ::1:5516...
    > * TCP_NODELAY set
    > * Connected to localhost (::1) port 5516 (#0)
    > > GET /webhooks/test_endpoint_get?token=tok3n HTTP/1.1
    > > Host: localhost:5516
    > > User-Agent: curl/7.66.0
    > > Accept: */*
    > >
    > * Mark bundle as not supporting multiuse
    > < HTTP/1.1 200 OK
    > < Date: Mon, 24 Feb 2020 15:56:54 GMT
    > < X-XSS-Protection: 1; mode=block
    > < X-Content-Type-Options: nosniff
    > < Vary: Accept-Encoding, User-Agent
    > < Content-Length: 0
    > <
    > * Connection #0 to host localhost left intact


## Custom event type: Chat Message

1. Open the `synthetic.xml` file and add the custom event type definition after the `UrlParameterTokenAuthentication` type:

```xml
<synthetic xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns="http://www.xebialabs.com/deployit/synthetic"
           xsi:schemaLocation="http://www.xebialabs.com/deployit/synthetic synthetic.xsd">

    <type type="mywebhooks.UrlParameterTokenAuthentication"
        extends="events.CustomJythonAuthentication"
        label="URL Parameter Token"
        description="Verify that the value of a given URL parameter matches the secret token.">
        <property name="parameter" kind="string" required="true"
            label="URL parameter"
            description="URL parameter name containing the token."/>
        <property name="token" kind="string" password="true" required="true"
            label="Token"
            description="Secret token value."/>
    </type>

    <type type="mywebhooks.ChatMessageEvent" extends="events.Event"
        label="Chat Message"
        description="A simple Chat Message event. A message sent by a user on a certain topic.">
        <property name="user" kind="string" required="true"/>
        <property name="topic" kind="string" required="true" default="#general"/>
        <property name="message" kind="string" required="true"/>
    </type>

</synthetic>
```

## Custom event source: Chat Message Event Source

1. As per ritual, let's open `synthetic.xml` to add this new definition for our Chat Message Event Source after the `ChatMessageEvent` type definition:

```xml
<synthetic xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns="http://www.xebialabs.com/deployit/synthetic"
           xsi:schemaLocation="http://www.xebialabs.com/deployit/synthetic synthetic.xsd">

    <type type="mywebhooks.UrlParameterTokenAuthentication"
        extends="events.CustomJythonAuthentication"
        label="URL Parameter Token"
        description="Verify that the value of a given URL parameter matches the secret token.">
        <property name="parameter" kind="string" required="true"
            label="URL parameter"
            description="URL parameter name containing the token."/>
        <property name="token" kind="string" password="true" required="true"
            label="Token"
            description="Secret token value."/>
    </type>

    <type type="mywebhooks.ChatMessageEvent" extends="events.Event"
        label="Chat Message"
        description="A simple Chat Message event. A message, sent by a user, on a certain topic.">
        <property name="user" kind="string" required="true"/>
        <property name="topic" kind="string" required="true" default="general"/>
        <property name="message" kind="string" required="true"/>
    </type>

    <type type="mywebhooks.ChatMessageEventSource" extends="events.CustomJythonEventSource"
        label="Chat Message Event Source"
        description="An Event Source emitting Chat Message Events. Consumes Http Request Events.">
        <property name="inputEventType" kind="string" default="events.HttpRequestEvent"
            required="true" hidden="true"/>
        <property name="outputEventType" kind="string" default="mywebhooks.ChatMessageEvent"
            required="true" hidden="true"/>
        <property name="scriptLocation" default="mywebhooks/ChatMessageEventSource.py" hidden="true" />
        <property name="topics" kind="set_of_string"
            required="false"
            label="Topics"
            description="Consider only messages on these topics. If left empty, consider all topics."/>
        <property name="ignoredUsers" kind="set_of_string"
            required="false"
            label="Ignore users"
            description="Ignore messages from these users."/>
        <property name="badWords" kind="set_of_string"
            required="false"
            label="Bad words"
            description="Ignore messages that include one of these words."/>

    </type>

</synthetic>
```

The `scriptLocation` property could have been omitted, since when left empty, it will try to find the script in the type's prefix (`mywebhooks`) directory, and will use the type's name (`ChatMessageEventSource`) plus `.py` as the filename, i.e. `mywebhooks/ChatMessageEventSource.py` in this example.

1. Let's write the `mywebhooks/ChatMessageEventSource.py` script:

```python
import json

global input
global config
global CI

# create the output event, given some data
def chat_message_event(data):
    return CI("mywebhooks.ChatMessageEvent", {
        'user': data['user'] if 'user' in data else None,
        'topic': data['topic'] if 'topic' in data else None,
        'message': data['message'] if 'message' in data else None
    })

# make sure all the properties of the chat message are present
def checkMessage(msg):
    return msg.user is not None and \
        msg.topic is not None and \
        msg.message is not None

# format message for logging
def formatMessage(msg):
    return '[#' + str(msg.topic) + '] ' + str(msg.user) + ': ' + str(msg.message)

# True iff 'topics' is not configured, empty, or if the topic's message is one of the 'topics'.
def topicFilter(msg):
    return config.topics is None or \
        len(config.topics) == 0 or \
        msg.topic in config.topics

# True iff 'ignoredUsers' is not configured, eepty, or if the user is NOT one of the 'ignoredUsers'.
def userFilter(msg):
    return config.ignoredUsers is None or \
        len(config.ignoredUsers) == 0 or \
        not (msg.user in config.ignoredUsers)

# True iff 'badWords' is not configured, empty, or if the message does not contain any of the 'badWords'.    
def badWordFilter(msg):
    if config.badWords is None or len(config.badWords) == 0:
        return True
    else:
        ok = True
        for bad in config.badWords:
            if bad in msg.message:
                ok = False
                break
        return ok

# supports both GET and POST HTTP endpoint for Webhooks event sources:
message_data = json.loads(input.content) if str(config.eventSource.method) == "POST" else input.parameters

# create the ChatMessageEvent
output = chat_message_event(message_data)

# check all properties of the message
if not checkMessage(output):
    print('Malformed chat message: ' + str(message_data))
    output = None
else:
    # apply the filters. if any of them fails, do not publish any event:
    if not topicFilter(output) or not userFilter(output) or not badWordFilter(output):
        print('Ignored! ' + formatMessage(output))
        output = None
    else:
        print(formatMessage(output))
```

Note that we didn't define a separate filter script here. We do both the parsing and the filtering at one time.

Not setting the `output` variable or setting it to `None` is equivalent to the filter script resulting in `accepted = False`.

### Test the new custom event source

#### Create the HTTP endpoint for webhooks

1. Add the types definitions from your `synthetic.xml` to your `XL_RELEASE_HOME/ext/synthetic.xml`
1. Copy the content of your `mywebhooks` directory to your `XL_RELEASE_HOME/ext/mywebhooks` directory
1. Restart XL Release
1. Under **Settings** > **Shared configuration** > **Webhooks and Event** create a new HTTP POST endpoint for webhooks
1. Name it, for example `My Test Endpoint (POST)`
1. Set the path, for example `test_endpoint_post`
1. Under authentication, select 'URL Parameter Token"
1. Configure the **parameter** field: type `token`
1. Configure the **token** field: type `tok3n`
1. Remember the endpoint's path
1. Save the HTTP endpoint for webhooks

#### Create the Chat Message Event Source

1. Under **Settings** > **Shared Configuration** > **Webhooks and Events**, create a new Chat Message Event Source
1. Name it, for example `My Chat Message Event Source (POST)`
1. Select the previously created POST endpoint as the **Event source**: `My Test Endpoint (POST)`
1. Add `general` and `commands` under **Topics**
1. Add `troll` and `banned_user` under **Ignored users**
1. Add `censored` and `swear` under **Bad words**
1. Save the `My Chat Message Event Source (POST)` event source

#### Create a test template and trigger
1. Under **Design** > **Folders**, create a new folder
1. Name it 'My Webhooks Folder'
1. Under **My Webhooks Folder** > **Templates**, create a new template
1. Name it 'My Chat Message Template'
1. Using the **Show** dropdown, select **Variables**
1. Add a new variable of type "Text" with key `user`. Ensure both `Required` and `Show on Create Release form` are enabled.
1. Add a new variable of type "Text" with key `topic`. Ensure both `Required` and `Show on Create Release form` are enabled.
1. Add a new variable of type "Text" with key `message`. Check the multiline checkbox. Ensure both `Required` and `Show on Create Release form` are enabled.
1. Using the **Show** dropdown, go back to **Release flow**
1. Add a manual task with title '[#${topic}] ${user}: ${message}'
1. Using the **Show** dropdown, select **Triggers**
1. Click the **Add trigger** button
1. Select **Webhook event trigger** for Trigger type
1. Name it, for example "My Chat Message Trigger (POST)"
1. Select the **Event source** `My Chat Message Event Source (POST)`
1. Select the **Event type** `Mywebhooks: Chat Message`
1. Enter the release title, for example `New Chat Message`
1. Under **Template variables** > **user**, click  and choose `User - user`
1. Under **Template variables** > **topic**, click  and choose `Topic - topic`
1. Under **Template variables** > **message**, click  and choose `Message - message`
1. Save the trigger

#### Verify that everything works as expected

1. The trigger should create a new release when a request is made to the correct endpoint, if it has good authentication, correct payload format and non-filtered content:

`curl -v \
    -X POST \
    -H 'Content-type: application/json' \
    'http://localhost:5516/webhooks/test_endpoint_post?token=tok3n' \
    -d '{"user": "me", "topic": "commands", "message": "Hello\nWorld"}'`

    > *   Trying ::1:5516...
    > * TCP_NODELAY set
    > * Connected to localhost (::1) port 5516 (#0)
    > > POST /webhooks/test_endpoint_post?token=tok3n HTTP/1.1
    > > Host: localhost:5516
    > > User-Agent: curl/7.66.0
    > > Accept: */*
    > > Content-type: application/json
    > > Content-Length: 62
    > >
    > * upload completely sent off: 62 out of 62 bytes
    > * Mark bundle as not supporting multiuse
    > < HTTP/1.1 200 OK
    > < Date: Tue, 25 Feb 2020 10:32:51 GMT
    > < X-XSS-Protection: 1; mode=block
    > < X-Content-Type-Options: nosniff
    > < Vary: Accept-Encoding, User-Agent
    > < Content-Length: 0
    > <
    > * Connection #0 to host localhost left intact

Check that a new release named "New Chat Message" was created.

The task title should read `[#commands] me: Hello\nWorld`

![New Chat Message](../images/webhooks/new-chat-message-release.png)

Complete or abort this release to avoid confusing it with another release when testing other scenarios.

1. It should not create a new release when a request is made to the correct endpoint, if it has good authentication, correct payload but on a topic that is not considered:

`curl -v \
    -X POST \
    -H 'Content-type: application/json' \
    'http://localhost:5516/webhooks/test_endpoint_post?token=tok3n' \
    -d '{"user": "me", "topic": "other", "message": "Hello\nWorld"}'`

    > *   Trying ::1:5516...
    > * TCP_NODELAY set
    > * Connected to localhost (::1) port 5516 (#0)
    > > POST /webhooks/test_endpoint_post?token=tok3n HTTP/1.1
    > > Host: localhost:5516
    > > User-Agent: curl/7.66.0
    > > Accept: */*
    > > Content-type: application/json
    > > Content-Length: 62
    > >
    > * upload completely sent off: 62 out of 62 bytes
    > * Mark bundle as not supporting multiuse
    > < HTTP/1.1 200 OK
    > < Date: Tue, 25 Feb 2020 10:32:51 GMT
    > < X-XSS-Protection: 1; mode=block
    > < X-Content-Type-Options: nosniff
    > < Vary: Accept-Encoding, User-Agent
    > < Content-Length: 0
    > <
    > * Connection #0 to host localhost left intact

Verify that no new releases were created.

1. It should not create a new release when a request is made to the correct endpoint, if it has good authentication, correct payload but is from an ignored user:

`curl -v \
    -X POST \
    -H 'Content-type: application/json' \
    'http://localhost:5516/webhooks/test_endpoint_post?token=tok3n' \
    -d '{"user": "troll", "topic": "commands", "message": "Hello\nWorld"}'`

    > *   Trying ::1:5516...
    > * TCP_NODELAY set
    > * Connected to localhost (::1) port 5516 (#0)
    > > POST /webhooks/test_endpoint_post?token=tok3n HTTP/1.1
    > > Host: localhost:5516
    > > User-Agent: curl/7.66.0
    > > Accept: */*
    > > Content-type: application/json
    > > Content-Length: 62
    > >
    > * upload completely sent off: 62 out of 62 bytes
    > * Mark bundle as not supporting multiuse
    > < HTTP/1.1 200 OK
    > < Date: Tue, 25 Feb 2020 10:32:51 GMT
    > < X-XSS-Protection: 1; mode=block
    > < X-Content-Type-Options: nosniff
    > < Vary: Accept-Encoding, User-Agent
    > < Content-Length: 0
    > <
    > * Connection #0 to host localhost left intact

Verify that no new releases were created.

1. It should not create a new release when a request is made to the correct endpoint, if it has good authentication, correct payload but contains a bad word:

`curl -v \
    -X POST \
    -H 'Content-type: application/json' \
    'http://localhost:5516/webhooks/test_endpoint_post?token=tok3n' \
    -d '{"user": "me", "topic": "commands", "message": "Hello\ncensored\nWorld"}'`

    > *   Trying ::1:5516...
    > * TCP_NODELAY set
    > * Connected to localhost (::1) port 5516 (#0)
    > > POST /webhooks/test_endpoint_post?token=tok3n HTTP/1.1
    > > Host: localhost:5516
    > > User-Agent: curl/7.66.0
    > > Accept: */*
    > > Content-type: application/json
    > > Content-Length: 62
    > >
    > * upload completely sent off: 62 out of 62 bytes
    > * Mark bundle as not supporting multiuse
    > < HTTP/1.1 200 OK
    > < Date: Tue, 25 Feb 2020 10:32:51 GMT
    > < X-XSS-Protection: 1; mode=block
    > < X-Content-Type-Options: nosniff
    > < Vary: Accept-Encoding, User-Agent
    > < Content-Length: 0
    > <
    > * Connection #0 to host localhost left intact

Verify that no new releases were created.

#### Exercise: verify that it works using a GET endpoint

1. Edit the `My Chat Message Event Source (POST)` event source
1. rename it to `My Chat Message Event Source (GET)`
1. select a GET endpoint for its source
1. save the event source

Send requests to the selected GET endpoint and verify that everything else still works as before - that is, releases are created when authentication is good and the chat message content passes the configured filters, as in the previous section of this tutorial.

Using curl, this would look something like:

`curl -v \
    -X GET \
    'http://localhost:5516/webhooks/test_endpoint_get?token=tok3n&user=me&topic=command&message=Hello%0AWorld'`

## Package your plugin

1. Go to your plugin's working directory (`xlr-mywebhooks-plugin`)
1. Go up one level: `cd ..`
1. Create a build directory: `rm -rf xlr-mywebhooks-plugin-build && mkdir xlr-mywebhooks-plugin-build`
1. Enter the build directory: `cd xlr-mywebhooks-plugin-build`
1. Add the `MANIFEST.MF` file: `mkdir -p META-INF && echo 'Manifest-Version: 1.0' > META-INF/MANIFEST.MF`
1. Copy the synthetic.xml file: `cp ../xlr-mywebhooks-plugin/synthetic.xml .`
1. Copy the scripts directory: `cp -r ../xlr-mywebhooks-plugin/mywebhooks .`
1. Create the plugin: `zip -r ../xlr-mywebhooks-plugin-0.1.jar .`
1. Exit the build directory: `cd ..`
1. Your plugin is ready in your current directory, named `xlr-mywebhooks-plugin-0.1.jar`
1. You can upload to your XL Release instance using the Plugin Manager screen (required being admin and a non-clustered installation).
1. Alternatively, place the `xlr-mywebhooks-plugin-0.1.jar` file in the `XL_RELEASE_HOME/plugin/__local__` directory
1. Remember to remove your type definitions from `XL_RELEASE_HOME/ext/synthetic.xml` and remove the `XL_RELEASE_HOME/ext/mywebhooks` directory from your installation used during development and testing once you install the plugin.
1. Restart XL Release
