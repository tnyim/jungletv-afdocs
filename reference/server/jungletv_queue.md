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
queue.addEventListener(type, listener)
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
queue.removeEventListener(type, listener)
```

##### Parameters

- `type` - A case-sensitive string corresponding to the [event type](#events) from which to unsubscribe.
- `listener` - The function previously passed to [`addEventListener()`](#addeventlistener), that should no longer be called whenever an event of the given `type` occurs.

##### Return value

None.

### `setEnqueuingPermission()`

Sets what users can add new entries to the media queue.
Applications may be able to enqueue media regardless of this setting.
To read the current value of this setting, use [`enqueuingPermission`](#enqueuingPermission).

#### Syntax

```js
queue.setEnqueuingPermission(permission)
queue.setEnqueuingPermission("enabled_password_required", password)
```

##### Parameters

- `permission` - A case-sensitive string determining the restriction applied to human-initiated enqueuing.
  The admissible values are the same as for the [`enqueuingPermission`](#enqueuingPermission) property.
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

If the application page is unpublished or the application is terminated, the queue entry will be removed.

Application page queue entries, if not set to unskippable (which can be achieved using the `options` object), may be skipped as any other queue entry would - assuming skipping is enabled at the time the corresponding queue entry is playing.

#### Syntax

```js
queue.enqueuePage(pageID, placement)
queue.enqueuePage(pageID, placement, length)
queue.enqueuePage(pageID, placement, length, options)
```

##### Parameters

- `pageID` - A case-sensitive string containing the ID of the application page to enqueue, as passed to [`publishFile()`](jungletv_pages.md#publishfile).
- `placement` - A case-sensitive [queue placement string](#queue-placement-string) containing the desired placement of the new queue entry.
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

### `enqueuingPermission`

This read-only property indicates which users can add new entries to the media queue.
Applications may be able to enqueue media regardless of this setting.
To modify this setting, use [`setEnqueuingPermission()`](#setenqueuingpermission) (a setter function is necessary because some modes require extra arguments).

The possible values are as follows:

| Enqueuing Permission String | Description                                                                                                                   |
| --------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| `enabled`                   | Everyone can add entries to the queue.                                                                                        |
| `enabled_staff_only`        | Only JungleTV staff, and users who staff have marked as VIP, can add entries to the queue.                                    |
| `enabled_password_required` | Only JungleTV staff, users who staff have marked as VIP, and users with knowledge of a password can add entries to the queue. |
| `disabled`                  | Nobody can add entries to the queue. The JungleTV UI will indicate this is due to upcoming maintenance.                       |

#### Syntax

```js
queue.enqueuingPermission = "enabled"
queue.enqueuingPermission = "enabled_staff_only"
queue.enqueuingPermission = "enabled_password_required"
queue.enqueuingPermission = "disabled"
```

### `entries`

This read-only property is an array of [queue entry objects](#queue-entry-object) representing the entries currently in the media queue, sorted in their current order.
The first entry is the currently playing entry.

#### Syntax

```js
queue.entries
```

### `playing`

This read-only property is a [queue entry object](#queue-entry-object) representing the currently playing queue entry.
It is `undefined` when no queue entry is currently playing.

#### Syntax

```js
queue.playing
```

### `length`

This read-only property represents the number of entries in the media queue.
It is functionally equivalent to `length` property of the [`entries`](#entries) array.

#### Syntax

```js
queue.length
```

### `lengthUpToCursor`

This read-only property represents the number of entries in the queue up to the insert cursor.
If no cursor is set, this property returns the same value as [`length`](#length).

```js
queue.lengthUpToCursor
```

### `removalOfOwnEntriesAllowed`

This writable boolean property controls whether the user who added a given entry to the queue is allowed to remove it.
Users are still subject to rate limits when removing their own entries, even when this setting is set to `true`.
Does not apply to staff or applications.

#### Syntax

```js
queue.removalOfOwnEntriesAllowed = true
queue.removalOfOwnEntriesAllowed = false
```

### `newQueueEntriesAllUnskippable`

This writable boolean property controls whether new queue entries are made unskippable at no additional cost, regardless of whether users request for them to be unskippable.

#### Syntax

```js
queue.newQueueEntriesAllUnskippable = true
queue.newQueueEntriesAllUnskippable = false
```

### `skippingAllowed`

This writable boolean property controls whether unprivileged users can use any forms of media skipping.
Does not affect entry self-removal, which is controlled by [`removalOfOwnEntriesAllowed`](#removalofownentriesallowed).

#### Syntax

```js
queue.skippingAllowed = true
queue.skippingAllowed = false
```

### `reorderingAllowed`

This writable boolean property controls whether users are able to reorder queue entries by spending JP.

#### Syntax

```js
queue.reorderingAllowed = true
queue.reorderingAllowed = false
```

### `insertCursor`

This writable string property allows for defining the queue insert cursor, i.e. the position at which entries are inserted in the queue when adding entries with placement `"later"`.
The property should be set to the ID of the media queue entry _below_ (i.e. at an higher index in `entries`) that where the cursor should appear.
Set to `null` or `undefined` to clear the cursor, causing new entries to be added to the end of the queue.

#### Syntax

```js
queue.insertCursor = "13b5cba1-416d-4db9-8407-645df98ac637"
queue.insertCursor = undefined
```

### `playingSince`

This read-only property is the Date since when the media queue has been playing non-stop.
It is `undefined` when no queue entry is currently playing.

#### Syntax

```js
queue.playingSince
```

### `crowdfunding`

Object containing properties and methods related to crowdfunded transactions ("Skip & Tip").
See [`crowdfunding` object](#pricing-object).

### `pricing`

Object containing properties and methods related to queue entry pricing.
See [`pricing` object](#pricing-object).

## Events

Listen to these events using [`addEventListener()`](#addeventlistener).

### `queueupdated`

This event is fired when the list of entries in the queue, or some of its associated settings, are updated.

#### Syntax

```js
queue.addEventListener("queueupdated", (event) => {})
```

#### Event properties

| Field  | Type   | Description                      |
| ------ | ------ | -------------------------------- |
| `type` | string | Guaranteed to be `queueupdated`. |

### `entryadded`

This event is fired when an entry is added to the queue.

#### Syntax

```js
queue.addEventListener("entryadded", (event) => {})
```

#### Event properties

| Field       | Type                               | Description                                                       |
| ----------- | ---------------------------------- | ----------------------------------------------------------------- |
| `type`      | string                             | Guaranteed to be `entryadded`.                                    |
| `entry`     | [Queue entry](#queue-entry-object) | The added entry.                                                  |
| `index`     | number                             | The position of the added entry in the queue.                     |
| `placement` | string                             | The requested type of [queue placement](#queue-placement-string). |

### `entrymoved`

This event is fired when an entry is moved in the queue.

#### Syntax

```js
queue.addEventListener("entrymoved", (event) => {})
```

#### Event properties

| Field           | Type                               | Description                                                                                                           |
| --------------- | ---------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| `type`          | string                             | Guaranteed to be `entrymoved`.                                                                                        |
| `entry`         | [Queue entry](#queue-entry-object) | The moved entry.                                                                                                      |
| `previousIndex` | number                             | The queue position occupied by the entry prior to being moved.                                                        |
| `currentIndex`  | number                             | The queue position presently occupied by the entry, after being moved.                                                |
| `user`          | **[[TODO] User]**                  | The user who moved the queue entry.                                                                                   |
| `direction`     | string                             | Whether the entry was moved `up` (closer to the currently playing entry) or `down` (to be played later in the queue). |

### `entryremoved`

This event is fired when an entry is removed from the queue.

#### Syntax

```js
queue.addEventListener("entryremoved", (event) => {})
```

#### Event properties

| Field         | Type                               | Description                                                                 |
| ------------- | ---------------------------------- | --------------------------------------------------------------------------- |
| `type`        | string                             | Guaranteed to be `entryremoved`.                                            |
| `entry`       | [Queue entry](#queue-entry-object) | The removed entry.                                                          |
| `index`       | number                             | The queue position occupied by the entry prior to being removed.            |
| `selfRemoval` | boolean                            | Whether the removal of the entry was requested by the user who enqueued it. |

### `mediachanged`

This event is fired when the currently playing media changes.

#### Syntax

```js
queue.addEventListener("mediachanged", (event) => {})
```

#### Event properties

| Field          | Type                               | Description                                 |
| -------------- | ---------------------------------- | ------------------------------------------- |
| `type`         | string                             | Guaranteed to be `mediachanged`.            |
| `playingEntry` | [Queue entry](#queue-entry-object) | The queue entry which just started playing. |

### `skippingallowedchanged`

This event is fired when the ability to skip entries is enabled or disabled.

#### Syntax

```js
queue.addEventListener("skippingallowedchanged", (event) => {})
```

#### Event properties

| Field  | Type   | Description                                |
| ------ | ------ | ------------------------------------------ |
| `type` | string | Guaranteed to be `skippingallowedchanged`. |

## Associated types

### Queue placement string

The different types of desired placement when enqueuing an entry are represented by the following strings:

| Queue Placement String | Description                                                                                                                                                                                                       |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `later`                | Used when the newly added queue entry is to be placed at the end of the queue, or wherever the insert cursor is placed.                                                                                           |
| `aftercurrent`         | Used when the newly added queue entry is to be placed after the currently playing entry.                                                                                                                          |
| `now`                  | Used when the newly added queue entry should replace any currently playing entry, skipping it. If the currently playing entry is unskippable or skipping is disabled, this placement behaves like `aftercurrent`. |

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