# App bridge API reference

This section contains a reference for the methods and properties exported by the [app bridge script](../../manual/pages.md#client-side-framework-appbridge).
All methods and properties documented in this reference can be accessed via the `window.appbridge` object that is present in contexts where the app bridge script was loaded successfully, as documented in the [parent section](./README.md).

## Methods

### `serverMethod()`

Makes a remote call to the application's server logic.

#### Syntax

```js
window.appbridge.serverMethod(methodName, arg1, /* ..., */ argN)
```

##### Parameters

- `methodName` - A case-sensitive string identifying the remote method to call, as passed to [`registerMethod()`](../server/jungletv_rpc.md#registermethod).
- This function accepts an indefinite number of additional parameters of arbitrary types, that will be used on the invocation of the registered method handler on the server.
  Arguments will be automatically serialized and deserialized using JSON.

##### Return value

A promise that will resolve to the value returned by the remote method handler, which may be none (`undefined`).
The return value will be automatically serialized and deserialized using JSON.

##### Exceptions

Exceptions thrown by this method (which may manifest as promise rejections when not using `await`) may be attributed to a number of underlying causes:

- Communication failure between the application page and the host JungleTV SPA, or between the JungleTV SPA and the JungleTV server, which may have been due to a network connectivity problem/timeout;
- The JungleTV AF application is stopped;
- A handler for the specified `methodName` has not been [`registered`](../server/jungletv_rpc.md#registermethod) by the application;
- The remote method handler threw an exception;
- The remote method handler took too long to return, resulting in a request timeout in the underlying transport.

### `emitToServer()`

Emits an event to the application's server logic.

#### Syntax

```js
window.appbridge.emitToServer(eventName, arg1, /* ..., */ argN)
```

##### Parameters

- `eventName` - A case-sensitive string identifying the event, as passed to [`addEventListener()`](../server/jungletv_rpc.md#addeventlistener).
- This function accepts an indefinite number of additional parameters of arbitrary types, that will be used on the invocation of the registered event handlers on the server.
  Arguments will be automatically serialized and deserialized using JSON.

##### Return value

None.

##### Exceptions

Exceptions thrown by this method may only be attributed to a communication failure between the application page and the host JungleTV SPA.
The call succeeds in all other cases, even if the communication between the JungleTV SPA hosting the page and the JungleTV server is severed.

### `navigateToApplicationPage()`

Instructs the JungleTV host page to navigate to a different application page, belonging to the same or a different application.

#### Syntax

```js
window.appbridge.navigateToApplicationPage(pageID)
window.appbridge.navigateToApplicationPage(pageID, applicationID)
```

##### Parameters

- `pageID` - A case-sensitive string identifying the application page, as passed to [`publishFile()`](../server/jungletv_pages.md#publishfile).
- `applicationID` - An optional case-sensitive string identifying the application to which the destination page belongs, to be used when the destination belongs to a different application.

##### Return value

None.

### `navigate()`

Instructs the JungleTV host page to navigate to a different JungleTV SPA route.

#### Syntax

```js
window.appbridge.navigate(path)
```

##### Parameters

- `path` - The route where to navigate, as will be passed to the [`navigate()` function of svelte-navigator](https://github.com/mefechoel/svelte-navigator#navigate).

##### Return value

None.

### `resolveApplicationFileURL()`

Resolves the URL that can be used to reference a public file of this application, within the context of the page.

#### Syntax

```js
window.appbridge.resolveApplicationFileURL(fileName)
```

##### Parameters

- `fileName` - The name of the public application file whose URL is to be resolved.

##### Return value

A promise that will resolve to a string containing the relative URL that can be used to reference the specified application file.

### `getApplicationID()`

Resolves the ID of the application to which the page being executed belongs.

#### Syntax

```js
window.appbridge.getApplicationID()
```

##### Parameters

None.

##### Return value

A promise that will resolve to a string containing the ID of the application to which the page belongs.

### `getApplicationVersion()`

Resolves the version of the application to which the page being executed belongs.

#### Syntax

```js
window.appbridge.getApplicationVersion()
```

##### Parameters

None.

##### Return value

A promise that will resolve to a string containing the version of the application to which the page belongs, represented as a number of milliseconds since midnight, January 1, 1970 UTC.
This value equals that obtained via [`process.versions.application`](../server/node_process.md#versions) in server-side code.

### `getApplicationPageID()`

Resolves the ID of the current application page.

#### Syntax

```js
window.appbridge.getApplicationPageID()
```

##### Parameters

None.

##### Return value

A promise that will resolve to a string containing the ID of the current application page, as previously passed to [`publishFile()`](../server/jungletv_pages.md#publishfile).

### `getApplicationPagePathname()`

Resolves the path name of the application page being executed, if the page is being rendered in `standalone` mode.

#### Syntax

```js
window.appbridge.getApplicationPagePathname()
```

##### Parameters

None.

##### Return value

A promise that will resolve to a string contianing the page path name, that is, the part of the containing page's path that follows the page ID.

### `getApplicationPageSearch()`

Resolves the "search" portion of the containing page's URL, if the page is being rendered in `standalone` mode.

#### Syntax

```js
window.appbridge.getApplicationPageSearch()
```

##### Parameters

None.

##### Return value

A promise that will resolve to a string contianing the `window.location.search` of the containing page.

### `getApplicationPageHash()`

Resolves the "hash" portion of the containing page's URL, if the page is being rendered in `standalone` mode.

#### Syntax

```js
window.appbridge.getApplicationPageHash()
```

##### Parameters

None.

##### Return value

A promise that will resolve to a string contianing the `window.location.hash` of the containing page.

### `alert()`

Shows an alert modal to the user.
This method should be used instead of the [Window `alert()` method](https://developer.mozilla.org/en-US/docs/Web/API/Window/alert).

The modal may not be opened immediately if a different modal is presently being displayed.

#### Syntax

```js
window.appbridge.alert(message)
window.appbridge.alert(message, title)
window.appbridge.alert(message, title, buttonLabel)
```

##### Parameters

- `message` - a string containing the main message of the modal.
- `title` - an optional string containing a title for the modal. Defaults to no title.
- `buttonLabel` - an optional string containing a custom label for the modal's only button.

##### Return value

A promise that will resolve to no value (`undefined`) as soon as the user closes the modal.

### `confirm()`

Shows a confirmation modal to the user.
This method should be used instead of the [Window `confirm()` method](https://developer.mozilla.org/en-US/docs/Web/API/Window/confirm).

The modal may not be opened immediately if a different modal is presently being displayed.

#### Syntax

```js
window.appbridge.confirm(question)
window.appbridge.confirm(question, title)
window.appbridge.confirm(question, title, positiveAnswerLabel)
window.appbridge.confirm(question, title, positiveAnswerLabel, negativeAnswerLabel)
```

##### Parameters

- `question` - a string containing the main message of the modal.
- `title` - an optional string containing a title for the modal.
  Defaults to no title.
- `positiveAnswerLabel` - an optional string containing a custom label for the confirmation acceptance button.
  Defaults to "Yes".
- `negativeAnswerLabel` - an optional string containing a custom label for the confirmation refusal button.
  Defaults to "No".

##### Return value

A promise that will resolve to a boolean that is true if the user accepted the confirmation, i.e. pressed the button labeled with the `positiveAnswerLabel` string.

### `prompt()`

Shows a text input modal to the user.
This method should be used instead of the [Window `prompt()` method](https://developer.mozilla.org/en-US/docs/Web/API/Window/prompt).

The modal may not be opened immediately if a different modal is presently being displayed.

#### Syntax

```js
window.appbridge.prompt(question)
window.appbridge.prompt(question, title)
window.appbridge.prompt(question, title, placeholder)
window.appbridge.prompt(question, title, placeholder, initialValue)
window.appbridge.prompt(question, title, placeholder, initialValue, positiveAnswerLabel)
window.appbridge.prompt(question, title, placeholder, initialValue, positiveAnswerLabel, negativeAnswerLabel)
```

##### Parameters

- `question` - a string containing the main message of the modal.
- `title` - an optional string containing a title for the modal.
  Defaults to no title.
- `placeholder` - an optional string containing the placeholder text that will be shown when the input is empty.
  Defaults to no placeholder.
- `initialValue` - an optional string containing the initial value for the text input.
  Defaults to an empty string.
- `positiveAnswerLabel` - an optional string containing a custom label for the confirmation acceptance button.
  Defaults to "OK".
- `negativeAnswerLabel` - an optional string containing a custom label for the confirmation refusal button.
  Defaults to "Cancel".

##### Return value

A promise that will resolve to a string containing the value of the text input, or to `null` if the user dismissed the modal (e.g. by pressing the button labeled with the `negativeAnswerLabel` string).

### `getUserAddress()`

Resolves the rewards address of the currently authenticated user.

#### Syntax

```js
window.appbridge.getUserAddress()
```

##### Parameters

None.

##### Return value

A promise that will resolve to a string containing the reward address of the user, or `undefined` if the user is not authenticated.

### `getUserPermissionLevel()`

Resolves the permission level of the user.

#### Syntax

```js
window.appbridge.getUserPermissionLevel()
```

##### Parameters

None.

##### Return value

A promise that will resolve to a string containing the permission level of the user:

- `unauthenticated` if the user is not authenticated;
- `user` if the user is authenticated with common user privileges;
- `admin` if the user is authenticated as a staff member.

### `showUserProfile()`

Shows a modal containing the profile of a user.
The modal may not be opened immediately if a different modal is presently being displayed.

#### Syntax

```js
window.appbridge.showUserProfile(userAddress)
```

##### Parameters

- `userAddress` - the reward address of the user whose profile should be shown.

##### Return value

None.

## Properties

### `BRIDGE_VERSION`

This read-only integer property indicates the version of the app bridge script.

#### Syntax

```js
window.appbridge.BRIDGE_VERSION
```

### `server`

This property is the object on which to register listeners for events coming from the application's server logic.

#### Syntax

```js
window.appbridge.server.addEventListener(eventName, eventHandler)
window.appbridge.server.removeEventListener(eventName, eventHandler)
```

Event handlers will be called with events whose `detail` field is an array containing the arguments passed to the [`emitToAll()`](../reference/server/jungletv_rpc.md#emittoall), [`emitToPage()`](../reference/server/jungletv_rpc.md#emittopage), [`emitToUser()`](../reference/server/jungletv_rpc.md#emittouser) and [`emitToPageUser()`](../reference/server/jungletv_rpc.md#emittopageuser) functions on the application's server code, which will automatically be serialized and deserialized using JSON.

### `page`

This property is the object on which to register listeners for events coming from the host JungleTV SPA.

#### Syntax

```js
window.appbridge.page.addEventListener(eventName, eventHandler)
window.appbridge.page.removeEventListener(eventName, eventHandler)
```

#### Events

**`connected`**

This event is fired when the connection between the host JungleTV SPA and the server-side application instance is established.

**`disconnected`**

This event is fired when the connection between the host JungleTV SPA and the server-side application instance is lost.

**`mounted`**

This event is fired when the host JungleTV SPA has finished loading the application page in its DOM.
The `detail` for this event is an object containing the following fields:

| Field  | Type   | Description                                                                                                                                 |
| ------ | ------ | ------------------------------------------------------------------------------------------------------------------------------------------- |
| `role` | string | `standalone`, `activity`, `sidebar` or `chatattachment` depending on the role played by the application page within the host JungleTV page. |

**`destroyed`**

This event is fired when the host JungleTV SPA is removing the application page from its DOM.