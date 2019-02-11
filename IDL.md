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
  boolean includeScheduled = false;
};
```
