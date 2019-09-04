# Notification Triggers

**Written**: 2019-01-28<br/>
**Updated**: 2019-02-11

Web developers have the ability to display notifications using the [Notifications API](https://notifications.spec.whatwg.org/). This ability is often used in collaboration with the [Push API](https://w3c.github.io/push-api/) to inform the user of time-sensitive information, such as a breaking news article or a received message. Notifications are then shown by allowing JavaScript code to run on the user's device.

**Notification Triggers** are a mechanism for preparing notifications that are to be shown when certain conditions are met. The _trigger_ could be time-based, location-based or otherwise—for this explainer, we'll focus on time-based triggers.

This makes it possible to prepare notifications to be displayed at a particular time without involving the server, and also improves reliability by removing the dependency on network connectivity. This can be essential for the user experience of certain types of web applications, for example calendars.

## Why do we care?
* The Push API is not reliable for triggering notifications which must be shown at a particular time. The ability to receive messages depends on the device's connectivity, and features such as _deep sleep_ may further delay delivery. An example where this is important are notifications reminding you of a 5 AM departure time to the airport.
* Unlike usage of the Push API, having notifications be prepared in advance helps browsers to avoid the cost of starting a Service Worker. Particularly on mobile devices, the impact of this can be significant.
* Not starting the Service Worker for prepared notifications has a privacy-preserving effect: the user's IP address won't be shared with the server by removing the need for network requests, reducing IP-to-location tracking.

## Goals
* Make it possible to prepare notifications that should be shown in the future by introducing time-based triggers.
* Create a mechanism for _defining a trigger_ that works for the Notifications API, but can be extended to other APIs.

## Non-goals
* Define a comprehensive set of possible triggers.

# Example code
Please see [this separate document](IDL.md) for the proposed WebIDL additions.

## Scheduling a reminder ten minutes before an appointment
```javascript
async function createAppointment(tag, title, timestamp) {
  // .. create the appointment in application-level code ..

  // Schedule a reminder to be shown to the user:
  await swRegistration.showNotification(title, {
    tag, body: 'Your appointment is due in ten minutes!',
    showTrigger: new TimestampTrigger(timestamp - TEN_MINUTES)
  });
}
```

## Canceling a previously scheduled reminder
```javascript
async function cancelAppointment(tag) {
  const notifications = await swRegistration.getNotifications({
    tag, includeTriggered: true
  });

  if (notifications && notifications.length >= 1)
    notifications[0].close();
}
```

# Design decision

## Using `showNotification()` and `getNotifications()`
The Notification API extends the `ServiceWorkerRegistration` interface [with two methods](https://notifications.spec.whatwg.org/#service-worker-api): `showNotification` and `getNotifications`. Particularly the former implies that something is going to happen _now_, which could lead to confusion.

Today our proposal builds on top of these APIs for simplicity, as can be seen in the [WebIDL proposal](IDL.md). An alternative would be to introduce new methods. These methods would likely share prototypes, interaction models and underlying algorithms, so the added value of doing this is unclear.

## Triggers or a `deliveryTime` property?
The introduction of the concept of Triggers makes this API easily extensible without changes. It also enables us to use the same concept for other APIs that would benefit of a delayed trigger point. In the case of time-based triggers we already identified at least one other type that triggers based on the local timezone of the device. This would allow certain use cases like alarm clocks and are easy to define as a new trigger type. Using a `deliveryTime` property would make adding use cases like this more confusing.

## Involving or excluding the Service Worker?
By excluding the Service Worker from the triggering part we can leverage optimizations on platforms like macOS, where we could use native APIs to delay showing the notifications, or on Android using the native task scheduling to reduce performance impact on the system. It is also reduces the chances of user tracking by not allowing network requests. The downside of this decision is that developers have to decide beforehand what content to show to the user. However, as this API is aimed at the offline experience, they would have to do this anyway because there there might not be an active network connection after defining the Notification.

# Privacy considerations

This API is only available in the context of a `ServiceWorkerRegistration`. This implies that all required data is stored in the same context and is automatically deleted when the ServiceWorker is deleted or the user deletes all site data for that origin. Blocking cookies also prevents ServiceWorkers, and therefore this API, from being used.

The only alternative to this API right now is to use the Push API. This allows sites to run JavaScript when receiving a push event and fetch additional data via the network. With Notification Triggers this is not the case anymore as all required data to display the notification has to be supplied beforehand. Showing a scheduled notification at the chosen time therefore doesn't require network connectivity, nor will cause any developer-observable effects.

# References and acknowledgements
* [Notifications API](https://notifications.spec.whatwg.org/)
* [Push API](https://w3c.github.io/push-api/)

Many thanks to @rknoll for the initial idea, and @jakearchibald for their ideas, input and discussion.
