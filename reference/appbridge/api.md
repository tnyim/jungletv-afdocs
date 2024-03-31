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

### `markdownToHTML()`

Parses JungleTV Flavored Markdown into HTML.
This corresponds to [GitHub Flavored Markdown](https://github.github.com/gfm/), with the following additions:
- Support for JungleTV-specific emotes - the right syntax for each emote can be obtained by typing the emote in the chat message composer, then selecting and copying it to the clipboard.
  Note that emotes may be added and removed over time;
- Support for Discord-style spoilers: `||text hidden by spoiler||`;
- Support for Discord-style timestamp formatting.
  The following formats are supported:

| Format example     | Result<sup>1</sup>                         | Notes        |
| ------------------ | ------------------------------------------ | ------------ |
| `<t:1708816943:d>` | 2/24/2024                                  |              |
| `<t:1708816943:D>` | February 24, 2024                          |              |
| `<t:1708816943:t>` | 11:22 PM                                   |              |
| `<t:1708816943:T>` | 11:22:23 PM                                |              |
| `<t:1708816943:f>` | February 24, 2024 at 11:22 PM              |              |
| `<t:1708816943:F>` | Saturday, February 24, 2024 at 11:22:23 PM |              |
| `<t:1708816943:R>` | 39 minutes ago                             | <sup>2</sup> |
| `<t:1708816943:C>` | 39 minutes and 21 seconds ago              | <sup>2</sup> |

<sup>1</sup> The result is locale-dependent; examples are shown for the en-US locale.

<sup>2</sup> The relative timestamp in the resulting markup is updated live, when added to the DOM of an application page where the appbridge is loaded.

#### Syntax

```js
window.appbridge.markdownToHTML(markdown)
```

##### Parameters

- `markdown` - A string containing the Markdown to parse.

##### Return value

A promise that will resolve to a string containing the resulting HTML markup.

### `limitedMarkdownToHTML()`

Parses a limited subset of JungleTV Flavored Markdown into HTML.
This is the same subset available for regular users to use in their chat messages.

#### Syntax

```js
window.appbridge.limitedMarkdownToHTML(markdown)
```

##### Parameters

- `markdown` - A string containing the Markdown to parse.

##### Return value

A promise that will resolve to a string containing the resulting HTML markup.

### `showNavigationBarNotification()`

Shows an ephemeral notification on the navigation bar.
If other notifications are presently showing, the new one will be enqueued to display after them.

#### Syntax

```js
window.appbridge.showNavigationBarNotification(message)
window.appbridge.showNavigationBarNotification(message, duration)
window.appbridge.showNavigationBarNotification(message, duration, href)
```

##### Parameters

- `message` - A string containing the message to show.
  Inline Markdown features will be formatted according to rules similar to those used with [`markdownToHTML()`](#markdowntohtml).
- `duration` - An optional number representing the length of time for which the notification should show, in milliseconds.
  Defaults to 7000.
  Must not be greater than 15000.
- `href` - An optional string containing an internal website link to navigate to when the user clicks the notification.

##### Return value

A promise that will resolve once the notification is enqueued for display.

### `getPlayerVolume()`

Resolves the current player volume preference being used by playing media.
This allows application pages to set the volume of their own sounds to the same one the user has selected for playing media.

#### Syntax

```js
window.appbridge.getPlayerVolume()
```

##### Parameters

None.

##### Return value

A promise that will resolve to the current volume preference as a fraction between 0 and 1.

### `setPlayerVolume()`

Sets the preference for the player volume to use by playing media.
Application pages may only use this method when mounted as playing media.
Together with [`getPlayerVolume()`](#getplayervolume), this allows application pages to implement volume sliders that correctly set the JungleTV SPA-wide user preference for media volume.

#### Syntax

```js
window.appbridge.setPlayerVolume(volume)
```

##### Parameters

 - `volume` - A number representing the volume preference to set, as a fraction between 0 and 1.

##### Return value

A promise that will resolve once the volume preference is set.

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