# `jungletv:ipc` module

The `jungletv:ipc` module allows for communication between the running JungleTV AF applications.

IPC stands for [Inter-process communication](https://en.wikipedia.org/wiki/Inter-process_communication).

This module is not imported by default. To use this module, import it in your server scripts as follows:

```js
const ipc = require("jungletv:ipc")
```

## Methods

### `addEventListener()`

Registers a function to be called whenever an application emits the event with the specified name to this application.
More than one listener may be registered to be called for each event type.
Other applications can pass arguments when they trigger an event, but it is not possible for the receiving application to directly return values, and the sending application is not notified about event delivery.
If an application is expecting to receive events from a specific application, it must check the `source` field of the [context object](#context-object) field to confirm the source.

#### Syntax

```js
ipc.addEventListener(eventName, listener)
```

##### Parameters

- `eventName` - A case-sensitive string identifying the event type.
- `listener` - A function that will be executed whenever this type of event is emitted by an application.
  The function will be called with at least one argument, a [context object](#context-object), followed by any arguments included by the sending application when emitting the event.

##### Return value

None.

### `removeEventListener()`

Ceases calling a function previously registered with [`addEventListener()`](#addeventlistener) whenever an event of the specified type is emitted by an application.

#### Syntax

```js
ipc.removeEventListener(eventName, listener)
```

##### Parameters

- `eventName` - A case-sensitive string identifying the event type.
- `listener` - The function previously passed to [`addEventListener()`](#addeventlistener), that should no longer be called whenever an event of the specified type occurs.

### `emitToApplication()`

Emits an event to the specified application.
This method does not wait for event delivery before returning.
Using this method alone, it is not possible to know whether the application received the event.

#### Syntax

```js
ipc.emitToApplication(applicationID, eventName)
ipc.emitToApplication(applicationID, eventName, arg1, /* ..., */ argN)
```

##### Parameters

- `applicationID` - A case-sensitive string representing the ID of the application to target.
  If this ID does not match the ID of a running application, the message is lost.
- `eventName` - A case-sensitive string identifying the event type.
  If this string does not match one for which the receiving application has [registered a listener](#addeventlistener), the message is lost.
- This function accepts an indefinite number of additional parameters of arbitrary types, that will be serialized using JSON and transmitted to the receiving application.

##### Return value

None.

## Associated types

### Context object

Represents the context of an inter-process event reception.

| Field    | Type   | Description                                             |
| -------- | ------ | ------------------------------------------------------- |
| `source` | string | ID of the application from which this event originated. |