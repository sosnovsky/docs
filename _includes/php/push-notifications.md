# Push Notifications

## Introduction

Push Notifications are a great way to keep your users engaged and informed about your app. You can reach your entire user base quickly and effectively. This guide will help you through the setup process and the general usage of Parse to send push notifications.

<div class='tip info'><div>
The PHP SDK does not currently support receiving pushes. It can only be used to send notifications to iOS and Android applications and to check the status of recent pushes.
</div></div>

## Setting Up Push

There is no setup required to use the PHP SDK for sending push notifications. If you haven't configured your [iOS](#setup/iOS) or [Android](#setup/Android) clients to use Push, take a look at their respective setup instruction using the platform toggle at the top.

## Installations

Every Parse application installed on a device registered for push notifications has an associated `Installation` object. The `Installation` object is where you store all the data needed to target push notifications. For example, in a baseball app, you could store the teams a user is interested in to send updates about their performance.

Note that `Installation` data can only be modified by the client SDKs, the data browser, or the REST API.

This class has several special fields that help you manage and target devices.

*   **`badge`**: The current value of the icon badge for iOS apps. Changes to this value on the server will be used for future badge-increment push notifications.
*   **`channels`**: An array of the channels to which a device is currently subscribed.
*   **`timeZone`**: The current time zone where the target device is located. This value is synchronized every time an `Installation` object is saved from the device _(readonly)_.
*   **`deviceType`**: The type of device, "ios" or "android" _(readonly)_.
*   **`installationId`**: Unique Id for the device used by Parse _(readonly)_.
*   **`deviceToken`**: The Apple generated token used for iOS devices _(readonly)_.

## Sending Pushes

There are two ways to send push notifications using Parse: [channels](#sending-channels/PHP) and [advanced targeting](#sending-queries/PHP). Channels offer a simple and easy to use model for sending pushes, while advanced targeting offers a more powerful and flexible model. Both are fully compatible with each other and will be covered in this section.

You can view your past push notifications on the Parse.com push console for up to 30 days after creating your push.  For pushes scheduled in the future, you can delete the push on the web console as long as no sends have happened yet. After you send the push, the web console shows push analytics graphs.

### Using Channels

The simplest way to start sending notifications is using channels. This allows you to use a publisher-subscriber model for sending pushes. Devices start by subscribing to one or more channels, and notifications can later be sent to these subscribers. The channels subscribed to by a given `Installation` are stored in the `channels` field of the `Installation` object.

#### Subscribing to Channels

The PHP SDK does not currently support subscribing iOS and Android devices for pushes. Take a look at the [iOS](#sending-channels/iOS), [Android](#sending-channels/Android) or [REST](#sending-channels/REST) Push guide using the platform toggle at the top.

#### Sending Pushes to Channels

With the PHP SDK, the following code can be used to alert all subscribers of the "Giants" and "Mets" channels about the results of the game. This will display a notification center alert to iOS users and a system tray notification to Android users.

<pre><code class="php">
$data = array("alert" => "Hi!");

ParsePush::send(array(
  "channels" => ["PHPFans"],
  "data" => $data
), true);
</code></pre>

### Using Advanced Targeting

While channels are great for many applications, sometimes you need more precision when targeting the recipients of your pushes. Parse allows you to write a query for any subset of your `Installation` objects using the [querying API](/docs/php_guide#queries) and to send them a push.

Since `Installation` objects are just like any other object stored in Parse, you can save any data you want and even create relationships between `Installation` objects and your other objects. This allows you to send pushes to a very customized and dynamic segment of your user base.

#### Saving Installation Data

The PHP SDK does not currently support modifying `Installation` objects. Take a look at the [iOS](#sending-queries/iOS), [Android](#sending-queries/Android) or [REST](#sending-queries/REST) Push guide using the platform toggle at the top.

#### Sending Pushes to Queries

Once you have your data stored on your `Installation` objects, you can use a query to target a subset of these devices. `Parse.Installation` queries work just like any other [Parse query](/docs/php_guide#queries).

<pre><code class="php">
$query = ParseInstallation::query();
$query->equalTo("design", "rad");
ParsePush::send(array(
  "where" => $query,
  "data" => $data
), true);
</code></pre>

We can even use channels with our query. To send a push to all subscribers of the "Giants" channel but filtered by those who want score update, we can do the following:

<pre><code class="php">
$query = ParseInstallation::query();
$query->equalTo("channels", "Giants");
$query->equalTo("scores", true);

ParsePush::send(array(
  "where" => $query,
  "data" => array(
    "alert" => "Giants scored against the A's! It's now 2-2."
  )
), true);
</code></pre>

If we store relationships to other objects in our `Installation` class, we can also use those in our query. For example, we could send a push notification to all users near a given location like this.

 <pre><code class="php">
// Find users near a given location
$userQuery = ParseUser::query();
$userQuery->withinMiles("location", $stadiumLocation, 1.0);

// Find devices associated with these users
$pushQuery = ParseInstallation::query();
$pushQuery->matchesQuery('user', $userQuery);

// Send push notification to query
ParsePush::send(array(
  "where" => $pushQuery,
  "data" => array(
    "alert" => "Free hotdogs at the Parse concession stand!"
  )
), true);
</code></pre>

## Sending Options

Push notifications can do more than just send a message. In iOS, pushes can also include the sound to be played, the badge number to display as well as any custom data you wish to send. In Android, it is even possible to specify an `Intent` to be fired upon receipt of a notification. An expiration date can also be set for the notification in case it is time sensitive.

### Customizing your Notifications

If you want to send more than just a message, you can set other fields in the `data` dictionary. There are some reserved fields that have a special meaning.

*   **`alert`**: the notification's message.
*   **`badge`**: _(iOS only)_ the value indicated in the top right corner of the app icon. This can be set to a value or to `Increment` in order to increment the current value by 1.
*   **`sound`**: _(iOS only)_ the name of a sound file in the application bundle.
*   **`content-available`**: _(iOS only)_ If you are a writing a [Newsstand](http://developer.apple.com/library/iOS/#technotes/tn2280/_index.html) app, or an app using the Remote Notification Background Mode [introduced in iOS7](https://developer.apple.com/library/ios/releasenotes/General/WhatsNewIniOS/Articles/iOS7.html#//apple_ref/doc/uid/TP40013162-SW10) (a.k.a. "Background Push"), set this value to 1 to trigger a background download.
*   **`category`**: _(iOS only)_ the identifier of the [`UIUserNotificationCategory`](https://developer.apple.com/library/prerelease/ios/documentation/UIKit/Reference/UIUserNotificationCategory_class/index.html#//apple_ref/occ/cl/UIUserNotificationCategory) for this push notification.
*   **`uri`**: _(Android only)_ an optional field that contains a URI. When the notification is opened, an `Activity` associated      with opening the URI is launched.
*   **`title`**: _(Android only)_ the value displayed in the Android system tray notification.

For example, to send a notification that increases the current badge number by 1 and plays a custom sound for iOS devices, and displays a particular title for Android users, you can do the following:

<pre><code class="php">
ParsePush::send(array(
  "channels" => [ "Mets" ],
  "data" => array(
    "alert" => "The Mets scored! The game is now tied 1-1.",
    "badge" => "Increment",
    "sound" => "cheering.caf",
    "title" => "Mets Score!"
  )
), true);
</code></pre>

It is also possible to specify your own data in this dictionary. As explained in the Receiving Notifications section for [iOS](#receiving/iOS) and [Android](#receiving/Android), iOS will give you access to this data only when the user opens your app via the notification and Android will provide you this data in the `Intent` if one is specified.

<pre><code class="php">
$query = ParseInstallation::query();
$query->equalTo('channels', 'Indians');
$query->equalTo('injuryReports', true);

ParsePush::send(array(
  "where" => $query,
  "data" => array(
    "action" => "com.example.UPDATE_STATUS"
    "alert" => "Ricky Vaughn was injured in last night's game!",
    "name" => "Vaughn",
    "newsItem" => "Man bites dog"
  )
), true);
</code></pre>

### Setting an Expiration Date

When a user's device is turned off or not connected to the internet, push notifications cannot be delivered. If you have a time sensitive notification that is not worth delivering late, you can set an expiration date. This avoids needlessly alerting users of information that may no longer be relevant.

There are two parameters provided by Parse to allow setting an expiration date for your notification. The first is `expiration_time` which takes a `DateTime` specifying when Parse should stop trying to send the notification.

Alternatively, you can use the `expiration_interval` parameter to specify a duration of time before your notification expires. This value is relative to the `push_time` parameter used to [schedule notifications](#scheduled/JavaScript). This means that a push notification scheduled to be sent out in 1 day and an expiration interval of 6 days can be received up to a week from now.

### Targeting by Platform

If you build a cross platform app, it is possible you may only want to target iOS or Android devices. There are two methods provided to filter which of these devices are targeted. Note that both platforms are targeted by default.

The following examples would send a different notification to Android and iOS users.

<pre><code class="php">
// Notification for Android users
$queryAndroid = ParseInstallation::query();
$queryAndroid->equalTo('deviceType', 'android');

ParsePush::send(array(
  "where" => $queryAndroid,
  "data" => array(
    "alert" => "Your suitcase has been filled with tiny robots!"
  )
), true);

// Notification for iOS users
$queryIOS = ParseInstallation::query();
$queryIOS->equalTo('deviceType', 'ios');

ParsePush::send(array(
  "where" => $queryIOS,
  "data" => array(
    "alert" => "Your suitcase has been filled with tiny apples!"
  )
), true);

// Notification for Windows 8 users
$queryWindows = ParseInstallation::query();
$queryWindows->equalTo('deviceType', 'winrt');

ParsePush::send(array(
  "where" => $queryWindows,
  "data" => array(
    "alert" => "Your suitcase has been filled with tiny surfaces!"
  )
), true);

// Notification for Windows Phone 8 users
$queryWP8 = ParseInstallation::query();
$queryWP8->equalTo('deviceType', 'winphone');

ParsePush::send(array(
  "where" => $queryWP8,
  "data" => array(
    "alert" => "Your suitcase is very hip; very metro."
  )
), true);
</code></pre>

## Scheduling Pushes

You can schedule a push in advance by specifying a `push_time` parameter of type `DateTime`.

If you also specify an `expiration_interval`, it will be calculated from the scheduled push time, not from the time the push is submitted. This means a push scheduled to be sent in a week with an expiration interval of a day will expire 8 days after the request is sent.

The scheduled time cannot be in the past, and can be up to two weeks in the future. It can be an ISO 8601 date with a date, time, and timezone, as in the example above, or it can be a numeric value representing a UNIX epoch time in seconds (UTC).

## Receiving Push Status

If the version of ParseServer you are running supports it you can retrieve a `PushStatus` object from the response.

From that you can fetch the status message of the push request, number of pushes sent, number failed, and more.

<pre><code class="php">
$data = array("alert" => "Hi!");

$response = ParsePush::send(array(
  "channels" => ["PHPFans"],
  "data" => $data
), true);

// check if a push status id is present
if(ParsePush::hasStatus($response)) {

    // Retrieve PushStatus object
    $pushStatus = ParsePush::getStatus($response);
    
    // get push status string
    $status = $pushStatus->getPushStatus();
    
    if($status == "succeeded") {
        // handle a successful push request
        
    } else if($status == "running") {
        // handle a running push request
    
    } else {
        // push request did not succeed
        
    }
        
    // get # pushes sent
    $sent = $pushStatus->getPushesSent();
    
    // get # pushes failed
    $failed = $pushStatus->getPushesFailed();
    
}
</code></pre>

## Receiving Pushes

The PHP SDK does not currently support receiving pushes. To learn more about handling received notifications in [iOS](#receiving/iOS) or [Android](#receiving/Android), use the platform toggle at the top.

## Troubleshooting

For tips on troubleshooting push notifications, check the troubleshooting sections for [iOS](#troubleshooting/iOS), [Android](#troubleshooting/Android), and [.NET](#troubleshooting/.NET) using the platform toggle at the top.
