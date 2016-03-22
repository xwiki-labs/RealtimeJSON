# RealtimeJSON API Specification

The purpose of this specification is to create a standard way to synchronize JSON data types (specifically lists, maps and strings) in realtime across multiple computers for collaborative document editing.

The means by which the RealtimeJSON object is created is is not part of the specification, it is specific to the implementation and it is where any configuration and setup must take place.

Examples of how a RealtimeJSON object might be created:

```javascript

CalebsRealtimeJSON.create("ws://path.to.websocket.server");

YannsRealtimeJSON.create(webRTCIntroducerObject);

someOtherAPI.getRealtimeJSON();

```

# RealtimeJSON Functions

## RealtimeJSON.getCollaborativeObject() -> Proxy<{}>

This function returns a proxy around an empty object, this proxy will intercept any changes to this object and will make proxies out of any objects which are pointed to from this object.

When a field of the object is set with another value, for example:

```javascript

const obj = RealtimeJSON.getCollaborativeObject();

const unsafeSubobject = { value: 1 };

const subObject = obj["key"] = unsafeSubobject;

```

subObject will be a proxy of unsafeSubobject, changes to subObject will be replicated but changes to unsafeSubobject will not.

As strings, booleans and numbers are immutible, this behavior applies only to Array and Object data types, use of RealtimeJSON with non-JSON data types such as undefined or typed arrays will cause an error.


# RealtimeJSON Events

## ready

Fired when the connection to the server/peers has been made and the RealtimeJSON object is ready to be edited.

```javascript

RealtimeJSON.on('ready', handler);

```

## change

Fired whenever a change comes in to the object or one of it's sub-objects. One change event will be fired for each object/subobject which has changed and an event handler can be called for each specific key with an argument newValue. If no keyPath is specified, the event handler will be called for each change in the object.

* **newValue** is the new value of the changed string, boolean, number, null, object or list.

```javascript

RealtimeJSON.on('change', handler);

RealtimeJSON.on('change', keyPath, handler);

```

* **keyPath** when specified, the event handler will only be applied when that specific key has changed.

Because of string immutibility it is considered that the string is replaced even if only a few characters in the string have changed.
