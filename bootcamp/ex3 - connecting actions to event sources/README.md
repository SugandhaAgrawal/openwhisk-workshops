# connecting actions to event sources

This exercise introduces concepts (triggers and rules) used by the platform to integrate external event providers.

*Once you have completed this exercise, you will have…*

- **Understood how event sources are integrated into the platform.**
- **Created example triggers and bound to actions using rules.**
- **Tested connecting triggers to external event sources.**

Once this exercise is finished, we can start to develop event-driven serverless applications using IBM Cloud Functions!

## Table Of Contents

* [Background](#background)
  * [Triggers](#triggers)
  * [Rules](#rules)
* [Creating Triggers](#creating-triggers)
* [Using Rules](#using-rules)
  * [Creating Rules](#creating-rules)
  * [Testing Rules](#testing-rules)
  * [Disabling Rules](#disabling-rules)
* [Connecting Trigger Feeds](#connecting-trigger-feeds)
* [EXERCISES](#exercises)

## Instructions

### Background

Serverless applications are often described as "event-driven" because you can connect serverless functions to external event sources, like message queues and database changes. When these external events fire, the serverless functions are automatically invoked, without any manual intervention.

In the previous example, we've been manually invoking actions using the command-line. Let's move onto connecting your actions to external event sources. OpenWhisk supports multiple external event sources like CouchDB, Apache Kafka, a cron-based scheduler and more.

Before we jump into the details, let's review some concepts which explain how this feature works in Apache OpenWhisk.

#### Triggers

Triggers are a named channel for a class of events. The following are examples of triggers:

- A trigger of location update events.
- A trigger of document uploads to a website.
- A trigger of incoming emails.

Triggers can be *fired* (activated) by using a dictionary of key-value pairs. Sometimes this dictionary is referred to as the *event*. As with actions, each firing of a trigger results in an activation ID.

Triggers can be explicitly fired by a user or by an external event source. A *feed* is a convenient way to configure an external event source to fire trigger events that can be consumed by IBM Cloud Functions. Examples of feeds include the following:

- CouchDB data change feed that fires a trigger event each time a document in a database is added or modified.
- A Git feed that fires a trigger event for every commit to a Git repository.

🎉🎉🎉 **Triggers are an implementation of the Observer pattern. Instances of triggers can be fired with parameters. Next we need to find out how to register observers. ** 🎉🎉🎉

#### Rules

A rule associates one trigger with one action, with every firing of the trigger causing the corresponding action to be invoked with the trigger event as input.

With the appropriate set of rules, it's possible for a single trigger event to invoke multiple actions, or for an action to be invoked as a response to events from multiple triggers.

For example, consider a system with the following actions:

- `classifyImage` action that detects the objects in an image and classifies them.
- `thumbnailImage` action that creates a thumbnail version of an image.

Also, suppose that there are two event sources that are firing the following triggers:

- `newTweet` trigger that is fired when a new tweet is posted.
- `imageUpload` trigger that is fired when an image is uploaded to a website.

You can set up rules so that a single trigger event invokes multiple actions, and have multiple triggers invoke the same action:

- `newTweet -> classifyImage` rule.
- `imageUpload -> classifyImage` rule.
- `imageUpload -> thumbnailImage` rule.

The three rules establish the following behavior: images in both tweets and uploaded images are classified, uploaded images are classified, and a thumbnail version is generated.

🎉🎉🎉 **Just remember rules allow you to register an observer on a trigger. That's all we need to know for now, let's look at using these new concepts… ** 🎉🎉🎉

### Creating Triggers

*Triggers* represent a named "channel" for a stream of events.

Let's create a trigger to send *location updates*:

```
$ ibmcloud fn trigger create locationUpdate
ok: created trigger locationUpdate
```

You can check that the trigger has been created like this:

```
$ ibmcloud fn trigger list
triggers
.../locationUpdate                         private
```

So far we have only created a named channel to which events can be fired.

Let's now fire the trigger by specifying its name and parameters:

```
$ ibmcloud fn trigger fire locationUpdate -p name "Donald" -p place "Washington, D.C"
ok: triggered /_/locationUpdate with id
```

Triggers also support default parameters. Firing this trigger without any parameters will pass in the default values.

```
$ ibmcloud fn trigger update locationUpdate -p name "Donald" -p place "Washington, D.C"
ok: updated trigger locationUpdate
$ ibmcloud fn trigger fire locationUpdate
ok: triggered /_/locationUpdate with id
```

Events you fire to the `locationUpdate` trigger currently do not do anything. To be useful, we need to create a rule that associates the trigger with an action.

🎉🎉🎉 **That was easy? Let's keep going by connecting actions to triggers…** 🎉🎉🎉

### Using Rules

Rules are used to associate a trigger with an action. Each time a trigger event is fired, the action is invoked with the event parameters.

#### Creating Rules

As an example, create a rule that calls the `hello` action whenever a location update is triggered.

1. Check the `hello` action exists and responds to the correct event parameters.

```
$ ibmcloud fn action invoke --result hello-js --param name Bernie --param place Vermont
{
    "payload": "Hello, Bernie from Vermont"
}
```

2. Check the trigger exists.

```
$ ibmcloud fn trigger get locationUpdate
ok: got trigger locationUpdate
{
    "name": "locationUpdate",
    "version": "0.0.1",
    "limits": {},
    "publish": false
}
```

3. Create the rule using the command-line. The three parameters are the name of the rule, the trigger, and the action.

```
$ ibmcloud fn rule create myRule locationUpdate hello-js
ok: created rule myRule
```

4. Retrieve rule details to show the trigger and action bound by this rule.

```
$ ibmcloud fn rule get myRule
ok: got rule myRule
{
    "namespace": "user@host.com_dev",
    "name": "myRule",
    "version": "0.0.1",
    "status": "active",
    "trigger": {
        "name": "locationUpdate",
        "path": "user@host.com_dev"
    },
    "action": {
        "name": "hello",
        "path": "user@host.com_dev"
    },
    "publish": false
}
```

#### Testing Rules

1. Fire the `locationUpdate` trigger. Each time that you fire the trigger with an event, the `hello` action is called with the event parameters.

```
$ ibmcloud fn trigger fire locationUpdate --param name Donald --param place "Washington, D.C."
ok: triggered /_/locationUpdate with id <activation id>
```

2. Verify that the action was invoked by checking the activations list.

```
$ ibmcloud fn activation list --limit 2
activations
<activation id 1> hello-js
<activation id 2> locationUpdate
```

We can see the trigger activation (`<activation id 1>`) is recorded, followed by the `hello-js` action activation (`<activation id 1>`).

3. Retrieving the trigger activation record will show the actions and rules invoked from this activation.

```
$ ibmcloud fn activation result <activation id 1>
{
   "payload": "Hello, Donald from Washington, D.C."
}
```

You can see that the hello action received the event payload and returned the expected string.

Activation records for triggers store the rules and actions fired for an event and the event parameters.

```
$ ibmcloud fn activation result <activation id 2>
{
    "name": "Donald",
    "place": "Washington, D.C."
}
$ ibmcloud fn activation logs <activation id 2>
{"statusCode":0,"success":true,"activationId":"<activation id 1>","rule":"user@host.com_dev/myRule","action":"user@host.com_dev/hello"}
```

You can create multiple rules that associate the same trigger with different actions. 

**Can you create another trigger and rule that calls the `hello` action? 🤔**

You can also use rules with sequences. For example, one can create an action sequence `recordLocationAndHello`that is activated by the rule `anotherRule`.

```
$ ibmcloud fn action create recordLocationAndHello --sequence /whisk.system/utils/echo,hello
$ ibmcloud fn rule create anotherRule locationUpdate recordLocationAndHello
```

#### Disabling Rules

Rules are enabled upon creation but can be disabled and re-enabled using the command-line. 

1. Disable the rule connecting the `locationUpdate` trigger and `hello` action.

```
$ ibmcloud fn rule disable myRule
```

2. Fire the trigger again.

```
$ ibmcloud fn trigger fire locationUpdate --param name Donald --param place "Washington, D.C."
ok: triggered /_/locationUpdate with id 
```

3. Check the activation list there are no new activation records.

```
$ ibmcloud fn activation list --limit 2
activations
5ee74025c2384f30a74025c2382f30c1 hello-js
5c153c01d76d49dc953c01d76d99dc34 locationUpdate
```

The latest activation records were from the previous example.

*Activation records for triggers are only recorded when they are bound to an active rule.*

🎉🎉🎉 **Right, now we have a way to connect actions to events in OpenWhisk, how do we connect triggers to event sources like messages queues? Enter trigger feeds…** 🎉🎉🎉

### Connecting Trigger Feeds

Trigger feeds allow you to connect triggers to external event sources. Event sources will fire registered triggers each time an event occurs. Here's a list of the event sources currently supported on IBM Cloud Functions: <https://github.com/apache/incubator-openwhisk/blob/master/docs/catalog.md>

This example shows how to use a feed in the [Alarms package](https://github.com/apache/incubator-openwhisk-package-alarms/blob/master/README.md) to fire a trigger every minute, which invokes an action using a rule.

1. Get a description of the feeds in the `/whisk.system/alarms` package.

```
$ ibmcloud fn package get --summary /whisk.system/alarms
package /whisk.system/alarms: Alarms and periodic utility
   (parameters: *apihost, *trigger_payload)
 feed   /whisk.system/alarms/interval: Fire trigger at specified interval
   (parameters: minutes, startDate, stopDate)
 feed   /whisk.system/alarms/once: Fire trigger once when alarm occurs
   (parameters: date, deleteAfterFire)
 feed   /whisk.system/alarms/alarm: Fire trigger when alarm occurs
   (parameters: cron, startDate, stopDate, timezone)
```

2. Retrieve the details for the `alarms/interval` feed.

```
$ ibmcloud fn action get --summary /whisk.system/alarms/interval
action /whisk.system/alarms/interval: Fire trigger at specified interval
   (parameters: *apihost, *isInterval, minutes, startDate, stopDate, *trigger_payload)
```

The `/whisk.system/alarms/interval` feed has the following parameters we need to pass in:

- `minutes`:  An integer representing the length of the interval (in minutes) between trigger fires.
- `trigger_payload`: The payload parameter value to set in each trigger event.

3. Create a trigger that fires every minute using this feed.

```
$ ibmcloud fn trigger create everyMinute --feed /whisk.system/alarms/interval -p minutes 1 -p trigger_payload "{\"name\":\"Mork\", \"place\":\"Ork\"}"
ok: invoked /whisk.system/alarms/interval with id <activation id>
...
ok: created trigger everyMinute
```

4. Connect this trigger to the `hello` action with a new rule.

```
$ ibmcloud fn rule create everyMinuteRule everyMinute hello-js
ok: created rule everyMinuteRule
```

5. Check that the action is being invoked every minute by polling for activation logs.

```
$ ibmcloud fn activation poll
Activation: 'hello-js' (<activation id 1>)
[]
Activation: 'everyMinute' (<activation id 2>)
[
    "{\"statusCode\":0,\"success\":true,\"activationId\":\"<activation id 1>\",\"rule\":\".../everyMinuteRule\",\"action\":\".../hello-js\"}"
]
```

You should see activations every minute the trigger and the action. The action receives the parameters `{"name":"Mork", "place":"Ork"}` on every invocation.

**PRETTY IMPORTANT: Let's delete the trigger and rule or this event will be running forever!**

```
$ ibmcloud fn trigger delete everyMinute
$ ibmcloud fn rule delete everyMinuteRule
```

🎉🎉🎉 **Understanding triggers and rules allows you to build event-driven applications on OpenWhisk. Create some actions, hook up events and let the platform take care of everything else, what could be easier?** 🎉🎉🎉

#### EXERCISES

Let's try out your new SERVERLESS SUPERPOWERS 💪 to build a real serverless application. 

***Can you use triggers & feeds to build a notification system when new users are added to a system?***

#### Background

Users are stored in a Cloudant database. When a new user is added to the database, we need to send them a welcome message. Users might provide an email address and/or phone number during registration. Welcome messages should be sent on all available channels.

#### Resources

IBM Cloud provides a free instance of the [Cloudant database](https://cloud.ibm.com/catalog/services/cloudant) to Lite account users.

Apache OpenWhisk supports listening to the [`changes` feed](http://docs.couchdb.org/en/2.0.0/api/database/changes.html) for a Cloudant database using this trigger feed: https://github.com/apache/incubator-openwhisk-package-cloudant

SendGrid [provides an API](https://sendgrid.com/docs/API_Reference/Web_API_v3/index.html) for sending emails. Twilio [provides an API](https://www.twilio.com/sms/api) for sending SMS messages.

Triggers can be fired programatically from an action using the [JavaScript client library.](https://github.com/apache/incubator-openwhisk-client-js#fire-trigger)

#### Architecture

This serverless application will have three actions: `user_changes`, `send_welcome_email` and `send_welcome_sms`. It will also have three triggers: `db_changes`, `new_user_email` and `new_user_sms`. `db_changes` will be connected to the Cloudant trigger feed.

`user_changes` will listen for database update using the `db_changes` trigger feed. When a new user record is received, it will check the record for email and phone number values. If either of those values exist, it will fire a custom trigger (`new_user_email` and `new_user_sms`) for each event type with the parameter value.

`send_welcome_email` will be connected to the `new_user_email` trigger. It will send a welcome email to the email address from the event.

`send_welcome_sms` will be connected to the `new_user_sms` trigger. It will send a welcome SMS message to the phone number from the event.

#### Tests

Test the email action by firing a trigger and checking for the email message.

```
$ ibmcloud fn trigger fire new_user_email -p address "blah@blah.com"
```

Test the SMS action by firing a trigger and checking for the SMS message.

```
$ ibmcloud fn trigger fire new_user_sms -p phone_number "XXXXX"
```

Test the DB listener action by firing a trigger with a sample database document. Provide a sample JSON document in the `doc.json` file.

```
$ ibmcloud fn trigger fire db_changes -P doc.jsonTest with correct password.
```

Create some new user documents in the database and with contact details and check the welcome messages are received.

#### Hint & Tips

Start with building & testing the sms and email actions. Once you have tested and verified they work, move onto the database listener action.

Don't create the trigger rules until you have verified all the actions work individually.