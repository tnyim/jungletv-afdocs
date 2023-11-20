# `jungletv:queue` module

The `jungletv:queue` module allows for interaction with the JungleTV queue subsystem.

This module is not imported by default. To use this module, import it in your server scripts as follows:

```js
const queue = require("jungletv:queue")
```

## Methods

### `addEventListener()`

Registers a function to be called whenever the specified [event](#events) occurs.
Depending on the event, the function may be invoked with arguments containing information about the event.
Refer to the documentation about each [event type](#events) for details.

#### Syntax

```js
chat.addEventListener(type, listener)
```

##### Parameters

- `type` - A case-sensitive string representing the [event type](#events) to listen for.
- `listener` - A function that will be called when an event of the specified type occurs.

##### Return value

None.

### `removeEventListener()`

Ceases calling a function previously registered with [`addEventListener()`](#addeventlistener) whenever the specified [event](#events) occurs.

#### Syntax

```js
chat.removeEventListener(type, listener)
```

##### Parameters

- `type` - A case-sensitive string corresponding to the [event type](#events) from which to unsubscribe.
- `listener` - The function previously passed to [`addEventListener()`](#addeventlistener), that should no longer be called whenever an event of the given `type` occurs.

##### Return value

None.

### `setEnqueuingPermission()`

Sets what users can add new entries to the media queue.
Applications may be able to enqueue media regardless of this setting.
To read the current value of this setting, use [`enqueuingPermission`](#enqueuingPermission) **[[TODO] confirm link correctness]**.

#### Syntax

```js
queue.setEnqueuingPermission(permission)
queue.setEnqueuingPermission("enabled_password_required", password)
```

##### Parameters

- `permission` - A case-sensitive string determining the restriction applied to human-initiated enqueuing.
  Possible values include:
  - `enabled`, which allows all users to enqueue.
  - `enabled_staff_only`, which exclusively allows JungleTV staff, and users who have been marked as VIP, to add entries to the queue.
  - `enabled_password_required`, which in addition to the users allowed by `enabled_staff_only`, also allows enqueuing by users who know a specific password.
  - `disabled`, which prevents anyone from enqueuing.
    The JungleTV UI will indicate this is due to upcoming maintenance.
- `password` - When `permission` is `enabled_password_required`, this must be a case-sensitive string containing the password that users must provide to be able to enqueue.

##### Return value

None.

### `removeEntry()`

Removes an entry from the queue.

#### Syntax

```js
queue.removeEntry(entryID)
```

##### Parameters

- `entryID` - A case-sensitive string containing the ID of the queue entry to remove.

##### Return value

A [queue entry object](#queue-entry-object) representing the removed entry.

### `moveEntry()`

Moves a queue entry to an adjacent position without costing the application JP.
Entries cannot be moved up when they are adjacent to the currently playing entry, or to the queue insert cursor if it is set.
Entries cannot be moved down when they are the last of the queue, or when they are adjacent to the queue insert cursor.

#### Syntax

```js
queue.moveEntry(entryID, direction)
```

##### Parameters

- `entryID` - A case-sensitive string containing the ID of the queue entry to move.
- `direction` - A case-sensitive string containing either `up` or `down`, depending on whether to move the queue entry closer to the currently playing entry or further away, respectively.

##### Return value

None.

### `moveEntryWithCost()`

Equivalent to [`moveEntry()`](#moveentry), but will deduct from the application JP balance and fail if the application does not have sufficient JP.

#### Syntax

```js
queue.moveEntryWithCost(entryID, direction)
```

##### Parameters

- `entryID` - A case-sensitive string containing the ID of the queue entry to move.
- `direction` - A case-sensitive string containing either `up` or `down`, depending on whether to move the queue entry closer to the currently playing entry or further away, respectively.

##### Return value

None.

### `enqueuePage()`

Enqueues an application page, to be "played" as if it were any other form of media.

The title of the created queue entry will default to the one passed to [`publishFile()`](jungletv_pages.md#publishfile), unless overridden via the `options` object.
The thumbnail of the created queue entry will default to a generic one, unless overridden via the `options` object.

Once the created queue entry reaches the top of the queue and begins "playing," the specified application page will be displayed on JungleTV clients in the same place where a media player normally goes.
The page will display alongside other homepage UI elements, including the sidebar (where an application page may also be displaying as a sidebar tab).
The page may also be displayed in a very small size, namely, whenever the user browses to other pages of the JungleTV SPA, as the media player will collapse to the bottom right corner of the screen until closed by the user, or until the user returns to the homepage.
Regardless of the size and placement of the application page, users will be able to interact with it, as they normally would if they had navigated to it.

Application page queue entries, if not set to unskippable (which can be achieved using the `options` object), may be skipped as any other queue entry would - assuming skipping is enabled at the time the corresponding queue entry is playing.

#### Syntax

```js
queue.enqueuePage(pageID, placement)
queue.enqueuePage(pageID, placement, length)
queue.enqueuePage(pageID, placement, length, options)
```

##### Parameters

- `pageID` - A case-sensitive string containing the ID of the application page to enqueue, as passed to [`publishFile()`](jungletv_pages.md#publishfile).
- `placement` - A case-sensitive string containing the desired placement of the new queue entry.
  Possible values include:
  - `later`, to add the new queue entry at the end of the queue, or wherever the insert cursor is placed.
  - `aftercurrent`, to place the new queue entry after the currently playing entry.
  - `now`, to replace any currently playing entry, skipping it.
    If the currently playing entry is unskippable or skipping is disabled, this behaves like `aftercurrent`.
- `length` - An optional number indicating the desired length, in milliseconds, for the new queue entry.
  If not specified or if set to infinity, once the corresponding queue entry begins playing, it will only stop once skipped/removed.
  A length for such queue entries will not be visible in the user interface.
  If a length is specified, it must be between one second and one hour, the period after which the media queue will automatically move to the next queue entry.
- `options` - An optional object containing additional options for the queue entry.
  The following properties are used:
  - `unskippable` - An optional boolean indicating whether the resulting queue entry may be skipped by the users.
    If set to true, the queue entry may only be skipped if it is removed by JungleTV staff or by an application.
  - `concealed` - An optional boolean indicating whether the resulting queue entry will hide its details before it begins playing.
    If set to true, the title, thumbnail and other information about the queue entry will not be revealed to unprivileged users, until it begins playing.
  - `title` - An optional string containing the desired title of the resulting queue entry.
    If not present, the title of the created queue entry will be the one passed to [`publishFile()`](jungletv_pages.md#publishfile).
  - `thumbnail` - An optional string containing the name of an application file which, when present, will override the thumbnail of the resulting queue entry.
    The file must be set to public and have an image file type.
    On the JungleTV clients, the image will be resized to fit the thumbnail area.
    Still, developers should be mindful not to provide images with unnecessarily large resolutions.
    If not present, a generic thumbnail will be used.

##### Return value

A [queue entry object](#queue-entry-object) representing the newly-created entry.

## Properties

**[[TODO] Complete]**

## Events

**[[TODO] Complete]**

## Associated types

**[[TODO] Complete]**

# `crowdfunding` object

The `crowdfunding` object of the `jungletv:queue` module allows for interaction with the JungleTV crowdfunding features ("Skip & Tip").

A reference to this object can be obtained as follows:

```js
const queue = require("jungletv:queue")
const crowdfunding = queue.crowdfunding
```

## Methods

**[[TODO] Complete]**

## Properties

**[[TODO] Complete]**

## Events

**[[TODO] Complete]**

## Associated types

**[[TODO] Complete]**

# `pricing` object

The `pricing` object of the `jungletv:queue` module allows for calculation and control of queue pricing.

A reference to this object can be obtained as follows:

```js
const queue = require("jungletv:queue")
const pricing = queue.pricing
```

## Methods

**[[TODO] Complete]**

## Properties

**[[TODO] Complete]**