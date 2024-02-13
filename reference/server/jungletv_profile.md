# `jungletv:profile` module

The `jungletv:profile` module allows for interaction with JungleTV user profiles.

This module is not imported by default. To use this module, import it in your server scripts as follows:

```js
const profile = require("jungletv:profile")
```

## Methods

### `getUser()`

Gets the user information for an arbitrary user, including their nickname.

#### Syntax

```js
profile.getUser(address)
```

##### Parameters

- `address` - Reward address of the user to fetch.

##### Return value

A [User](./common_types.md#user-object) object containing the information for the specified user.
Note that a User object will be returned for any valid Banano address, even those that have never signed in to the service.

### `getProfile()`

Gets the profile information for an arbitrary user.

#### Syntax

```js
profile.getProfile(address)
```

##### Parameters

- `address` - The reward address of the user whose profile should be fetched.

##### Return value

A promise that will resolve to a [user profile](#user-profile-object) object containing the biography and featured media of the user.
An empty profile object will be returned for valid Banano addreses that have never signed in to the service.

### `setProfileFeaturedMedia()`

Sets the featured media on the profile of an arbitrary user.
Note that this function will change the profile of any valid Banano address, even those that are yet to sign in to the service.

#### Syntax

```js
profile.setProfileFeaturedMedia(address, featuredMediaID)
```

##### Parameters

- `address` - The reward address of the user whose profile should be updated.
- `featuredMediaID` - The ID of the [media performance](./jungletv_queue.md#media-performance-object) to set as featured media, or `undefined` to clear the featured media.

##### Return value

A promise that will resolve when the update is complete.

### `setProfileBiography()`

Sets the biography on the profile of an arbitrary user.
Note that this function will set the biography of any valid Banano address, even those that are yet to sign in to the service.

#### Syntax

```js
profile.setProfileBiography(address, biography)
```

##### Parameters

- `address` - The reward address of the user whose profile should be updated.
- `biography` - The new biography for the user, which can be up to 512 characters long. May be an empty string to clear the biography.

##### Return value

A promise that will resolve when the update is complete.

### `clearProfile()`

Clears the biography and featured media on the profile of an arbitrary user.

#### Syntax

```js
profile.clearProfile(address)
```

##### Parameters

- `address` - The reward address of the user whose profile should be cleared.

##### Return value

A promise that will resolve when the profile is cleared.

### `setUserNickname()`

Sets the nickname of an arbitrary user, visible in messages sent by the user in chat as well as in leaderboards and other contexts.
The nickname is subject to similar restrictions as if it were set by the user.
It must be between 3 and 16 characters long, cannot look like a Banano address, and will be subject to sanitization before being defined.
Note that this function will set the nickname associated with any valid Banano address, even those that are yet to sign in to the service.

#### Syntax

```js
profile.setUserNickname(address, nickname)
```

##### Parameters

- `address` - The reward address of the user whose nickname should be updated.
- `nickname` - The new nickname for the user. When set to `null`, `undefined` or the empty string, the user will appear in chat using its rewards address.

##### Return value

A promise that will resolve when the nickname is changed.

### `getStatistics()`

Fetches statistics about a user since a certain point in time.

#### Syntax

```js
profile.getStatistics(address, since)
```

##### Parameters

- `address` - The reward address of the user to fetch statistics for.
- `since` - A Date representing the start of the time range for which to fetch statistics.

##### Return value

A promise that will resolve to a [profile statistics](#profile-statistics-object) object containing the user statistics since the specified point in time.

## Associated types

### User profile object

Contains extra fields about the user profile that are not present in the [User](./common_types.md#user-object) type.

| Field             | Type    | Description                                                                                                                                                                                                                                                              |
| ----------------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `biography`       | string  | The biography of the user, set by them or via the [setProfileBiography()](#setprofilebiography) method. May be the empty string if the user has not set a biography.                                                                                                     |
| `featuredMediaID` | string? | ID of the [media performance](./jungletv_queue.md#media-performance-object) that is featured on the user profile, set by them or via the [setProfileFeaturedMedia()](#setprofilefeaturedmedia) method. May be `undefined` if the profile does not have a featured media. |

### Profile statistics object

Contains statistics about a user, relative to a certain period in time.

| Field                  | Type   | Description                                                                                                |
| ---------------------- | ------ | ---------------------------------------------------------------------------------------------------------- |
| `spentRequesting`      | string | The amount, in raw Banano units, spent by the user in requesting media entries which have started playing. |
| `spentCrowdfunding`    | string | The amount, in raw Banano units, spent by the user in crowdfunding transactions (skipping or tipping).     |
| `spent`                | string | The sum, in raw Banano units, of `spentRequesting` and `spentCrowdfunding`.                                |
| `withdrawn`            | string | The amount, in raw Banano units, withdrawn by the user.                                                    |
| `mediaRequestCount`    | number | The number of media performances requested by the user, which have started playing.                        |
| `mediaRequestPlayTime` | number | The total play duration, in milliseconds, of the `mediaRequestCount` performances requested by the user.   |