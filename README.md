# Notification Triggers

**Written**: 2019-01-28<br/>
**Updated**: 2019-01-28

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

