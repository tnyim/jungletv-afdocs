# `jungletv:spectators` module

The `jungletv:spectators` module allows for obtaining information about the connected spectators (users connected to the media player) and managing the state of their media consumption sessions.

This module is not imported by default. To use this module, import it in your server scripts as follows:

```js
const spectators = require("jungletv:spectators")
```

## Methods

### `addEventListener()`

Registers a function to be called whenever the specified [event](#events) occurs.
Depending on the event, the function may be invoked with arguments containing information about the event.
Refer to the documentation about each [event type](#events) for details.

#### Syntax

```js
spectators.addEventListener(type, listener)
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
spectators.removeEventListener(type, listener)
```

##### Parameters

- `type` - A case-sensitive string corresponding to the [event type](#events) from which to unsubscribe.
- `listener` - The function previously passed to [`addEventListener()`](#addeventlistener), that should no longer be called whenever an event of the given `type` occurs.

##### Return value

None.

### `getSpectator()`

Obtains information about the spectator with the given reward address.

#### Syntax

```js
spectators.getSpectator(address)
```

##### Parameters

- `address` - A case-sensitive string containing the reward address of the user whose spectator data to fetch.

##### Return value

The [spectator object](#spectator-object) relative to the user, if the user is presently registered in the system as a spectator (which may happen even if they are no longer connected, but have disconnected recently), or `null` if the user is not presently registered as a spectator.

### `markAsActive()`

Marks a spectator as active, exempting them from activity checks for a typical minimum period of 15 minutes, starting from the moment this method is called.
This method should only be called if the user has very recently performed an action that can be reasonably attributed to sentient human activity.
This method has no effect if the spectator already has a pending activity challenge, or if the user is not registered as a spectator.

#### Syntax

```js
spectators.markAsActive(address)
```

##### Parameters

- `address` - The reward address of the user to mark as active.

##### Return value

None.

## Properties

### `connectedCount`

This read-only property contains the number of currently connected spectators.

#### Syntax

```js
spectators.connectedCount
```

### `eligibleEstimate`

This read-only property contains the estimated number of connected spectators currently eligible to receive rewards.

> See also [`computeEnqueuePricing()`](./jungletv_queue.md#computeenqueuepricing) for a method that estimates prices for media rewards based on this estimate and other values.

#### Syntax

```js
spectators.eligibleEstimate
```

## Events

Listen to these events using [`addEventListener()`](#addeventlistener).

### `rewardsdistributed`

This event is fired when rewards are distributed among eligible spectators.

#### Syntax

```js
spectators.addEventListener("rewardsdistributed", (event) => {})
```

#### Event properties

| Field              | Type                                                              | Description                                                                                                                     |
| ------------------ | ----------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| `type`             | string                                                            | Guaranteed to be `rewardsdistributed`.                                                                                          |
| `rewardBudget`     | string                                                            | The total reward amount distributed among eligible spectators, in raw Banano units, excluding any tips for the media requester. |
| `requesterReward`  | string                                                            | The amount distributed to the media requester.                                                                                  |
| `rewardedUsers`    | array of [User](./common_types.md#user-object)s                   | The eligible spectators rewarded, not including the media requester, who may have been rewarded depending on `requesterReward`. |
| `mediaPerformance` | [Media performance](./jungletv_queue.md#media-performance-object) | The media performance the spectators were rewarded for consuming.                                                               |

### `spectatorconnected`

This event is fired when a spectator connects to the media player.

#### Syntax

```js
spectators.addEventListener("spectatorconnected", (event) => {})
```

#### Event properties

| Field       | Type                           | Description                            |
| ----------- | ------------------------------ | -------------------------------------- |
| `type`      | string                         | Guaranteed to be `spectatorconnected`. |
| `spectator` | [Spectator](#spectator-object) | The spectator that has connected.      |

### `spectatordisconnected`

This event is fired when a spectator disconnects from the media player.

#### Syntax

```js
spectators.addEventListener("spectatordisconnected", (event) => {})
```

#### Event properties

| Field       | Type                           | Description                               |
| ----------- | ------------------------------ | ----------------------------------------- |
| `type`      | string                         | Guaranteed to be `spectatordisconnected`. |
| `spectator` | [Spectator](#spectator-object) | The spectator that has disconnected.      |

### `spectatoractivitychallenged`

This event is fired when a spectator is issued an activity challenge.

#### Syntax

```js
spectators.addEventListener("spectatoractivitychallenged", (event) => {})
```

#### Event properties

| Field                          | Type                           | Description                                                                                                                                                          |
| ------------------------------ | ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `type`                         | string                         | Guaranteed to be `spectatoractivitychallenged`.                                                                                                                      |
| `spectator`                    | [Spectator](#spectator-object) | The spectator to which the challenge has been issued.                                                                                                                |
| `hadPreviousUnsolvedChallenge` | boolean                        | Whether the spectator was yet to solve a challenge that had previously been issued to them in the current session.                                                   |
| `challengeDifficulty`          | string                         | The difficulty of the activity challenge that was issued: `"easy"` or `"hard"`, in the latter case, the challenge is guaranteed to contain a captcha-like component. |

### `spectatorsolvedactivitychallenge`

This event is fired when a spectator-submitted solution to an activity challenge has been processed.

#### Syntax

```js
spectators.addEventListener("spectatorsolvedactivitychallenge", (event) => {})
```

#### Event properties

| Field                 | Type                           | Description                                                                                                                                                                 |
| --------------------- | ------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `type`                | string                         | Guaranteed to be `spectatoractivitychallenged`.                                                                                                                             |
| `spectator`           | [Spectator](#spectator-object) | The spectator who solved the challenge.                                                                                                                                     |
| `challengedFor`       | number                         | The duration, in milliseconds, between the moment the spectator was challenged and the moment they submitted a solution to the challenge.                                   |
| `correctSolution`     | boolean                        | Whether the solution submitted by the spectator was considered correct.                                                                                                     |
| `challengeDifficulty` | string                         | The difficulty of the activity challenge that was solved: `"easy"` or `"hard"`, in the latter case, the challenge is guaranteed to have contained a captcha-like component. |

## Associated types

### Spectator object

Represents the media consumption session of a user that is or was recently connected to the media player.

| Field                     | Type                                                              | Description                                                                                                                                     |
| ------------------------- | ----------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| `user`                    | [User](./common_types.md#user-object)                             | The user entity associated with this spectator.                                                                                                 |
| `connectedAt`             | Date                                                              | The moment at which this spectator was registered.                                                                                              |
| `disconnectedAt`          | Date                                                              | The moment at which the last remaining connection of this user was destroyed, or `undefined` if the user is presently connected.                |
| `connectionCount`         | number                                                            | The number of active media player connections of this user.                                                                                     |
| `failedLegitimacyCheckAt` | Date?                                                             | The moment at which this spectator failed a legitimacy check, or `undefined` if the user is presently considered legitimate.                    |
| `activityChallengedAt`    | Date?                                                             | The moment at which this spectator was issued an activity challenge, or `undefined` if the user has no pending activity challenge.              |
| `remoteAddress`           | [Remote address information](#remote-address-information-object)? | Information about the remote network address of this spectator or `undefined` if no information is presently available.                         |
| `markAsActive()`          | function                                                          | When called, marks this spectator as active. Equivalent to calling [`markAsActive()`](#markasactive) with the reward address of this spectator. |

### Remote address information object

Contains information about the remote network address of a spectator.

| Field       | Type    | Description                                                                                                                                                           |
| ----------- | ------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `reputable` | boolean | Whether the remote address is considered to have sufficiently good reputation to be eligible for receiving rewards.                                                   |
| `asNumber`  | number? | [Autonomous system](https://en.wikipedia.org/wiki/Autonomous_system_(Internet)) number associated with the remote address. May be undefined if the number is unknown. |