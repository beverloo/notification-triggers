# Proposed WebIDL

## Definition of Trigger and TimestampTrigger
```webidl
[Exposed=(Window,Worker), SecureContext]
interface Trigger {
  readonly attribute USVString type;
};

[Constructor(DOMTimeStamp timestamp), Exposed=(Window,Worker), SecureContext]
interface TimestampTrigger : Trigger {
  readonly attribute DOMTimeStamp timestamp;
};
```

## Additions to the Notification API
```webidl
partial dictionary NotificationOptions {
  Trigger showTrigger = null;
};

partial dictionary GetNotificationOptions {
  boolean includeTriggered = false;
};

partial interface Notification {
  [SameObject] readonly attribute Trigger showTrigger;
}
```

The [create a notification](https://notifications.spec.whatwg.org/#create-a-notification) algorithm will be amended to throw a `TypeError` exception when a `showTrigger` has been provided, but not a `serviceWorkerRegistration`. This removes the ability to use _notification triggers_ for non-persistent notifications, which inherently are tied to the lifetime of the document. It will also throw a `TypeError` exception when a `TimestampTrigger` is provided that is more than 1 year ahead of now, calculated via `Date.now() + MAX_TRIGGER_FUTURE > timestamp` where `MAX_TRIGGER_FUTURE` is defined as `367 * 24 * 60 * 60 * 1000`. 367 days are chosen to account for leap years and seconds.
