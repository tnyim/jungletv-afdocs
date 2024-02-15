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
  - `baseReward` - An optional string containing the minimum reward, as a Banano raw amount, that will be paid to active spectators by the time the resulting queue entry finishes playing.
    This reward may be increased by the community while the queue entry is playing, via the crowdfunded tipping feature.
    The specified amount is debited from the application's [wallet](jungletv_wallet.md). If the wallet has insufficient funds, enqueuing will fail.
    By default, enqueued media entries do not have a base reward.
    [`computeEnqueuePricing()`](#computeenqueuepricing) can be used to compute a base reward amount that takes into account the current queue conditions.
  - `title` - An optional string containing the desired title of the resulting queue entry.
    If not present, the title of the created queue entry will be the one passed to [`publishFile()`](jungletv_pages.md#publishfile).
  - `thumbnail` - An optional string containing the name of an application file which, when present, will override the thumbnail of the resulting queue entry.
    The file must be set to public and have an image file type.
    On the JungleTV clients, the image will be resized to fit the thumbnail area.
    Still, developers should be mindful not to provide images with unnecessarily large resolutions.
    If not present, a generic thumbnail will be used.

##### Return value

A promise that will resolve to a [queue entry object](#queue-entry-object) representing the newly-created entry.

### `getPlayHistory()`

Retrieves the play history for media performances that took place between two dates, with results sorted by the order in which they played.

#### Syntax

```js
queue.getPlayHistory(since, until)
queue.getPlayHistory(since, until, options)
```

##### Parameters

- `since` - A Date representing the start of the time range for which retrieve play history.
- `until` - A Date representing the end of the time range for which to retrieve play history.
- `options` - An optional object containing additional options for the play history request.
  The following properties are used:
  - `filter` - An optional string indicating a filter for the results.
    This filter will match performance IDs exactly, and media titles inexactly.
    The inexact matching algorithm is implementation-specific - not documented and subject to change.
    This field powers the "search" function in the user-facing Play History page.
  - `descending` - An optional boolean which, when set to true, will sort results in descending order instead of the default ascending order.
  - `includeDisallowed` - An optional boolean which, when set to true, will include media that is presently disallowed on the service in the results.
    By default, presently disallowed media is excluded.
  - `includePlaying` - An optional boolean which, when set to true, will include the currently playing performance on the results.
    By default, the presently ongoing performance is excluded.
  - `limit` - An optional number which, when set to a positive integer, will set the maximum number of results to return.
  - `offset` - An optional number which, when set to a positive integer, will exclude the first results from the return value, up to the specified offset, to then return up to `limit` results.

##### Return value

A promise that will resolve to an array of [media performance objects](#media-performance-object) representing the play history for the specified period.

### `getEnqueueHistory()`

Retrieves the enqueuing history for past media performances that were enqueued between two dates, with results sorted by the order in which they were enqueued.
Does not include entries which never began playing or which are yet to begin playing.

This function is unable to retrieve entries which were enqueued before December 23, 2021.

#### Syntax

```js
queue.getEnqueueHistory(since, until)
queue.getEnqueueHistory(since, until, options)
```

##### Parameters

- `since` - A Date representing the start of the time range for which retrieve enqueue history.
- `until` - A Date representing the end of the time range for which to retrieve enqueue history.
- `options` - An optional object containing additional options for the enqueue history request.
  The following properties are used:
  - `filter` - An optional string indicating a filter for the results.
    This filter will match performance IDs exactly, and media titles inexactly.
    The inexact matching algorithm is implementation-specific - not documented and subject to change.
  - `descending` - An optional boolean which, when set to true, will sort results in descending order instead of the default ascending order.
  - `includeDisallowed` - An optional boolean which, when set to true, will include media that is presently disallowed on the service in the results.
    By default, presently disallowed media is excluded.
  - `includePlaying` - An optional boolean which, when set to true, will include the currently playing performance on the results.
    By default, the presently ongoing performance is excluded.
  - `limit` - An optional number which, when set to a positive integer, will set the maximum number of results to return.
  - `offset` - An optional number which, when set to a positive integer, will exclude the first results from the return value, up to the specified offset, to then return up to `limit` results.

##### Return value

A promise that will resolve to an array of [media performance objects](#media-performance-object) representing the enqueue history for the specified period, without including entries which never began playing or which are still in the media queue, yet to begin playing.

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
The cursor cannot be set to the currently playing queue entry.
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
See [`crowdfunding` object](#crowdfunding-object).

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

| Field           | Type                                  | Description                                                                                                           |
| --------------- | ------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| `type`          | string                                | Guaranteed to be `entrymoved`.                                                                                        |
| `entry`         | [Queue entry](#queue-entry-object)    | The moved entry.                                                                                                      |
| `previousIndex` | number                                | The queue position occupied by the entry prior to being moved.                                                        |
| `currentIndex`  | number                                | The queue position presently occupied by the entry, after being moved.                                                |
| `user`          | [User](./common_types.md#user-object) | The user who moved the queue entry.                                                                                   |
| `direction`     | string                                | Whether the entry was moved `up` (closer to the currently playing entry) or `down` (to be played later in the queue). |

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

| Field          | Type                                | Description                                                                           |
| -------------- | ----------------------------------- | ------------------------------------------------------------------------------------- |
| `type`         | string                              | Guaranteed to be `mediachanged`.                                                      |
| `playingEntry` | [Queue entry](#queue-entry-object)? | The queue entry which just started playing, or `undefined` if the queue became empty. |

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

### Queue entry object

Represents an entry that is, will be or has been in the media queue.
Extends the [media performance object](#media-performance-object).

| Field            | Type                                   | Description                                                                                                                                                                                                                                                                                            |
| ---------------- | -------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `id`             | string                                 | The globally unique identifier of this queue entry. Can be used to refer to this queue entry even after it is done playing.                                                                                                                                                                            |
| `media`          | [Media info](#media-info-object)       | Information about the media of this queue entry.                                                                                                                                                                                                                                                       |
| `requestedAt`    | Date                                   | Moment when this entry was added to the queue.                                                                                                                                                                                                                                                         |
| `requestedBy`    | [User](./common_types.md#user-object)? | The user who added this entry to the queue. May be `undefined` in the case of entries automatically enqueued by JungleTV or by staff.                                                                                                                                                                  |
| `requestCost`    | string                                 | How much the requester of this entry spent to enqueue this entry, in raw Banano units.                                                                                                                                                                                                                 |
| `unskippable`    | boolean                                | Whether this queue entry may be skipped by unprivileged users or through community skipping.                                                                                                                                                                                                           |
| `concealed`      | boolean                                | Whether the media information will only be visible to unprivileged users once this entry begins playing.                                                                                                                                                                                               |
| `movedBy`        | array of strings                       | List of the addresses of the users who moved this queue entry.                                                                                                                                                                                                                                         |
| `startedAt`      | Date?                                  | Moment when this entry began playing. May be `undefined` in the case of entries which are yet to begin playing.                                                                                                                                                                                        |
| `playedFor`      | number                                 | Duration in milliseconds corresponding to how long this queue entry has played.                                                                                                                                                                                                                        |
| `playing`        | boolean                                | Whether this queue entry is currently playing.                                                                                                                                                                                                                                                         |
| `played`         | boolean                                | Whether this queue entry has finished playing, in which case `playedFor` will not increase further.                                                                                                                                                                                                    |
| `remove()`       | function                               | When called, removes the entry from the queue. Equivalent to calling [removeEntry()](#removeentry) with the `id` of this entry.                                                                                                                                                                        |
| `move()`         | function                               | When called, move the queue entry to an adjacent position without costing the application JP. Equivalent to calling [moveEntry()](#moveentry) with the `id` of this entry. Takes one string argument, the direction, which may be `"up"` or `"down"`.                                                  |
| `moveWithCost()` | function                               | Equivalent to `move`, but will deduct from the application JP balance and fail if the application does not have sufficient JP. Equivalent to calling [moveEntryWithCost()](#moveentrywithcost) with the `id` of this entry. Takes one string argument, the direction, which may be `"up"` or `"down"`. |

### Media performance object

Information about one past, present or future performance of a [media](#media-info-object) on the service, which may take the form of a [queue entry](#queue-entry-object).

| Field         | Type                                   | Description                                                                                                                                                                       |
| ------------- | -------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `id`          | string                                 | The globally unique identifier of this performance, matches the identifier of the respective queue entry. Can be used to refer to this performance even after it is done playing. |
| `media`       | [Media info](#media-info-object)       | Information about the media that was performed.                                                                                                                                   |
| `requestedAt` | Date?                                  | Moment when this entry was added to the queue. May be undefined for performances that were enqueued before December 23, 2021.                                                     |
| `requestedBy` | [User](./common_types.md#user-object)? | The user who added this entry to the queue. May be `undefined` in the case of entries automatically enqueued by JungleTV or by staff.                                             |
| `requestCost` | string                                 | How much the requester of this performance spent to enqueue it, in raw Banano units.                                                                                              |
| `unskippable` | boolean                                | Whether this performance may be skipped by unprivileged users or through community skipping.                                                                                      |
| `startedAt`   | Date?                                  | Moment when this performance began. May be `undefined` in the case of performances which are yet to begin.                                                                        |
| `playedFor`   | number                                 | Duration in milliseconds corresponding to how long this performance lasted or for how long it has been ongoing.                                                                   |
| `playing`     | boolean                                | Whether this performance is currently ongoing.                                                                                                                                    |
| `played`      | boolean                                | Whether this performance has finished, in which case `playedFor` will not increase further.                                                                                       |

### Media info object

Information about the media associated with a queue entry.

| Field    | Type   | Description                                                                                                                                                                                                                                                     |
| -------- | ------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `length` | number | Length of the media in milliseconds - just the duration that is meant to play on the service.                                                                                                                                                                   |
| `offset` | number | Offset from the start of the media, in milliseconds, at which playback should start. For an underlying media with a total duration of 10 minutes and where the requester wishes to play from 5:00 to 7:00, `offset` will be 300000 and `length` will be 120000. |
| `title`  | string | Title of the media.                                                                                                                                                                                                                                             |
| `id`     | string | Media provider-specific unique identifier for the underlying media.                                                                                                                                                                                             |
| `type`   | string | Type (media provider) of the media. May be one of `yt_video`, `sc_track`, `document` or `app_page`.                                                                                                                                                             |

### Queue placement string

The different types of desired placement when enqueuing an entry are represented by the following strings:

| Queue Placement String | Description                                                                                                                                                                                                       |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `later`                | Used when the newly added queue entry is to be placed at the end of the queue, or wherever the insert cursor is placed.                                                                                           |
| `aftercurrent`         | Used when the newly added queue entry is to be placed after the currently playing entry.                                                                                                                          |
| `now`                  | Used when the newly added queue entry should replace any currently playing entry, skipping it. If the currently playing entry is unskippable or skipping is disabled, this placement behaves like `aftercurrent`. |

# `crowdfunding` object

The `crowdfunding` object of the `jungletv:queue` module allows for interaction with the JungleTV crowdfunding features ("Skip & Tip").

A reference to this object can be obtained as follows:

```js
const queue = require("jungletv:queue")
const crowdfunding = queue.crowdfunding
```

## `crowdfunding` Methods

### `addEventListener()`

Registers a function to be called whenever the specified [crowdfunding event](#crowdfunding-events) occurs.
Depending on the event, the function may be invoked with arguments containing information about the event.
Refer to the documentation about each [event type](#crowdfunding-events) for details.

#### Syntax

```js
queue.crowdfunding.addEventListener(type, listener)
```

##### Parameters

- `type` - A case-sensitive string representing the [event type](#crowdfunding-events) to listen for.
- `listener` - A function that will be called when an event of the specified type occurs.

##### Return value

None.

### `removeEventListener()`

Ceases calling a function previously registered with [`addEventListener()`](#addeventlistener-1) whenever the specified [crowdfunding event](#crowdfunding-events) occurs.

#### Syntax

```js
queue.crowdfunding.removeEventListener(type, listener)
```

##### Parameters

- `type` - A case-sensitive string representing the [event type](#crowdfunding-events) from which to unsubscribe.
- `listener` - The function previously passed to [`addEventListener()`](#addeventlistener-1), that should no longer be called whenever an event of the given `type` occurs.

##### Return value

None.

## `crowdfunding` Properties

### `skippingEnabled`

This writable boolean property controls whether crowdfunded skipping is enabled.
Crowdfunded skipping may be unavailable even when this setting is set to `true`, subject to other restrictions (e.g. [skippingAllowed](#skippingallowed)).

#### Syntax

```js
queue.crowdfunding.skippingEnabled = true
queue.crowdfunding.skippingEnabled = false
```

### `skipping`

This read-only property returns an object representing the current [crowdfunded skipping status](#crowdfunded-skipping-status-object).

#### Syntax

```js
queue.crowdfunding.skipping
```

### `tipping`

This read-only property returns an object representing the current [crowdfunded tipping status](#crowdfunded-tipping-status-object).

#### Syntax

```js
queue.crowdfunding.tipping
```

## `crowdfunding` Events

Listen to these events using the crowdfunding-specific [`addEventListener()`](#addeventlistener-1).

### `statusupdated`

This event is fired when the skipping or tipping statuses are updated.

#### Syntax

```js
queue.crowdfunding.addEventListener("statusupdated", (event) => {})
```

#### Event properties

| Field      | Type                                                               | Description                                             |
| ---------- | ------------------------------------------------------------------ | ------------------------------------------------------- |
| `type`     | string                                                             | Guaranteed to be `statusupdated`.                       |
| `skipping` | [Crowdfunded skipping status](#crowdfunded-skipping-status-object) | The current status of the crowdfunded skipping feature. |
| `tipping`  | [Crowdfunded tipping status](#crowdfunded-tipping-status-object)   | The current status of the crowdfunded tipping feature.  |

### `skipthresholdreductionmilestonereached`

This event is fired when a skip threshold reduction milestone is reached.

#### Syntax

```js
queue.crowdfunding.addEventListener("skipthresholdreductionmilestonereached", (event) => {})
```

#### Event properties

| Field             | Type   | Description                                                                      |
| ----------------- | ------ | -------------------------------------------------------------------------------- |
| `type`            | string | Guaranteed to be `skipthresholdreductionmilestonereached`.                       |
| `ratioOfOriginal` | number | Fraction of the original skip threshold that has been reached in this milestone. |

### `skipped`

This event is fired when currently playing entry is skipped via crowdfunding.

#### Syntax

```js
queue.crowdfunding.addEventListener("skipped", (event) => {})
```

#### Event properties

| Field     | Type   | Description                                                                     |
| --------- | ------ | ------------------------------------------------------------------------------- |
| `type`    | string | Guaranteed to be `skipped`.                                                     |
| `balance` | string | Amount, in raw Banano units, that the community paid to skip the playing entry. |

### `transactionreceived`

This event is fired when a transaction is received in the crowdfunded skipping or tipping accounts.

#### Syntax

```js
queue.crowdfunding.addEventListener("transactionreceived", (event) => {})
```

#### Event properties

| Field         | Type    | Description                                                                                                                                   |
| ------------- | ------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| `type`        | string  | Guaranteed to be `transactionreceived`.                                                                                                       |
| `txHash`      | string  | Block hash of the received transaction.                                                                                                       |
| `fromAddress` | string  | Address of the sender of the received transaction.                                                                                            |
| `amount`      | string  | Amount, in raw Banano units, that was received in this transaction.                                                                           |
| `receivedAt`  | Date    | Time at which this transaction was received.                                                                                                  |
| `txType`      | string  | `"skip"` or `"tip"`, depending on whether this was a crowdfunded skipping or a crowdfunded tipping transaction.                               |
| `forMedia`    | string? | Unique `id` of the [queue entry](#queue-entry-object) that was playing at the time of the transaction, or `undefined` if nothing was playing. |

## `crowdfunding` Associated types

### Crowdfunded skipping status object

Information about the crowdfunded skipping feature.

| Field                | Type                                                             | Description                                                                                                                                                                       |
| -------------------- | ---------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `status`             | [Crowdfunded skipping state](#crowdfunded-skipping-state-string) | State of the crowdfunded skipping feature, indicating whether it is presently possible for the community to skip, or the reason why not. See below for a list of possible values. |
| `address`            | string                                                           | Address of the crowdfunded skipping account.                                                                                                                                      |
| `balance`            | string                                                           | Balance, in raw Banano units, of the crowdfunded skipping account.                                                                                                                |
| `threshold`          | string                                                           | Balance of the crowdfunded skipping account, in raw Banano units, at which skipping will occur. May be influenced via [`crowdfundedSkipMultiplier`](#crowdfundedskipmultiplier).  |
| `thresholdLowerable` | boolean                                                          | Whether users are able to spend JP in order to decrease the `threshold`.                                                                                                          |

#### Crowdfunded skipping state string

A string indicating the status of the community skipping feature.

| State                              | Meaning                                                                                                                                                           |
| ---------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `possible`                         | Crowdfunded skipping is possible: the currently playing entry will be skipped as soon as the balance of the crowdfunded skipping account reaches the `threshold`. |
| `impossible_unskippable`           | Crowdfunded skipping is impossible because the currently playing queue entry is unskippable.                                                                      |
| `impossible_end_of_media_period`   | Crowdfunded skipping is impossible because we are near the end of the currently playing queue entry.                                                              |
| `impossible_no_media`              | Crowdfunded skipping is impossible because there is no currently playing queue entry.                                                                             |
| `impossible_unavailable`           | Crowdfunded skipping is unavailable for technical reasons.                                                                                                        |
| `impossible_disabled`              | The crowdfunded skipping feature is disabled (e.g. via [`skippingEnabled`](#skippingenabled)).                                                                    |
| `impossible_start_of_media_period` | Crowdfunded skipping is impossible because we are at the beginning of the currently playing queue entry.                                                          |


### Crowdfunded tipping status object

| Field     | Type   | Description                                                       |
| --------- | ------ | ----------------------------------------------------------------- |
| `address` | string | Address of the crowdfunded tipping account.                       |
| `balance` | string | Balance, in raw Banano units, of the crowdfunded tipping account. |

# `pricing` object

The `pricing` object of the `jungletv:queue` module allows for calculation and control of queue pricing.

A reference to this object can be obtained as follows:

```js
const queue = require("jungletv:queue")
const pricing = queue.pricing
```

## `pricing` Methods

### `computeEnqueuePricing()`

Compute the current pricing for a new queue entry, which would be requested to a user as a requirement for enqueuing at different placements.

The prices depend on what is being enqueued as well as the length of the media queue and the number of active spectators at the time of the call.

The costs may be adjusted via [`finalMultiplier`](#finalmultiplier) and [`minimumMultiplier`](#minimummultiplier).

#### Syntax

```js
queue.pricing.computeEnqueuePricing(length, unskippable, concealed)
```

##### Parameters

- `length` - Number representing the length of the media section in milliseconds.
- `unskippable` - A boolean indicating whether the entry is to be unskippable.
- `concealed` - A boolean indicating whether media information should be concealed until the media starts playing.

##### Return value

An [enqueue pricing object](#enqueue-pricing-object) containing the minimum required amount for enqueuing at different queue placements.

## `pricing` Properties

### `finalMultiplier`

This writable integer property represents the general multiplier applied to the cost of enqueuing.
It cannot be set lower than 1.

#### Syntax

```js
queue.pricing.finalMultiplier = 42
```

### `minimumMultiplier`

This writable integer property represents the the minimum prices multiplier, which sets a lower bound on the cost of enqueuing, in an attempt to ensure that all users get some reward regardless of the conditions at the time an entry plays.
It cannot be set lower than 20.

#### Syntax

```js
queue.pricing.minimumMultiplier = 42
```

### `crowdfundedSkipMultiplier`

This writable integer property represents the multiplier applied to the cost of crowdfunded skipping.
It cannot be set lower than 1.
It takes effect once the [state](#crowdfunded-skipping-state-string) of crowdfunded skipping changes, which may be provoked by toggling the availability of crowdfunded skipping (via [`skippingEnabled`](#skippingenabled) or the more general [`skippingAllowed`](#skippingallowed)).

#### Syntax

```js
queue.pricing.crowdfundedSkipMultiplier = 42
```

## `pricing` Associated types

### Enqueue pricing object

Represents the minimum required amounts for enqueuing at different [queue placements](#queue-placement-string).

| Field          | Type   | Description                                                                                            |
| -------------- | ------ | ------------------------------------------------------------------------------------------------------ |
| `later`        | string | The minimum amount, in raw Banano units, required to enqueue the entry using `later` placement.        |
| `aftercurrent` | string | The minimum amount, in raw Banano units, required to enqueue the entry using `aftercurrent` placement. |
| `now`          | string | The minimum amount, in raw Banano units, required to enqueue the entry using `now` placement.          |