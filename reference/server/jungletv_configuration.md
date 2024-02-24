# `jungletv:configuration` module

The `jungletv:configuration` module allows for altering different aspects of JungleTV's presentation and behavior.

This module is not imported by default. To use this module, import it in your server scripts as follows:

```js
const configuration = require("jungletv:configuration")
```

## Methods

### `setAppName()`

Defines a custom website name to be used in place of "JungleTV".

The change is immediately reflected on all connected media-consuming clients, and is automatically undone when the application terminates.

Multiple JAF applications can request to override this configuration.
In such cases, the reflected value will be that of the application that most recently requested to override the configuration, and which is yet to terminate or cease overriding the configuration value.

#### Syntax

```js
configuration.setAppName(name)
```

##### Parameters

- `name` - The name to temporarily use for the JungleTV web application.
  Must not be longer than 140 bytes when encoded using UTF-8.
  When set to `null`, `undefined` or the empty string, the AF application will stop overriding the JungleTV web application name.

##### Return value

In circumstances where the AF runtime is working as expected, this function will return a `true` boolean.

### `setAppLogo()`

Defines a custom website logo to be used in place of the default one.
The image to use must be an application file that has the Public property set and has an image MIME type.

The change is immediately reflected on all connected media-consuming clients, and is automatically undone when the application terminates.

Multiple JAF applications can request to override this configuration.
In such cases, the reflected value will be that of the application that most recently requested to override the configuration, and which is yet to terminate or cease overriding the configuration value.

#### Syntax

```js
configuration.setAppLogo(filename)
```

##### Parameters

- `filename` - The name of the application file to serve as the JungleTV website logo.
  This file must have the Public property enabled and have an image MIME type.
  When set to `null`, `undefined` or the empty string, the AF application will stop overriding the JungleTV website logo.

##### Return value

In circumstances where the JAF runtime is working as expected, this function will return a `true` boolean.

### `setAppFavicon()`

Defines a custom website favicon to be used in place of the default one.
The image to use must be an application file that has the Public property set and has an image MIME type.

The change is immediately reflected on all connected media-consuming clients, and is automatically undone when the application terminates.

Multiple JAF applications can request to override this configuration.
In such cases, the reflected value will be that of the application that most recently requested to override the configuration, and which is yet to terminate or cease overriding the configuration value.

#### Syntax

```js
configuration.setAppFavicon(filename)
```

##### Parameters

- `filename` - The name of the application file to serve as the JungleTV website favicon.
  This file must have the Public property enabled and have an image MIME type.
  When set to `null`, `undefined` or the empty string, the JAF application will stop overriding the JungleTV website favicon.

##### Return value

In circumstances where the AF runtime is working as expected, this function will return a `true` boolean.

### `setSidebarTab()`

Sets an application page, registered with [publishFile()](./jungletv_pages.md#publishfile), to be shown as an additional sidebar tab on the JungleTV homepage.
The tab's initial title will be the default title passed to [publishFile()](./jungletv_pages.md#publishfile) when publishing the page.
When the page makes use of the [App bridge](/reference/appbridge/), its document title will be automatically synchronized with the tab title, **while the tab is visible/selected**.
When not selected, the tab **may** retain the most recent title until it is reopened or removed, **or** it **may** revert to the page's default title.

Currently, application sidebar tabs can't be popped out of the main JungleTV application window like built-in tabs can (e.g. by middle-clicking on the tab title).

The new sidebar tab becomes immediately available (but not immediately visible, i.e. the selected sidebar tab will not change) on all connected media-consuming clients, and is automatically removed when the application terminates or when the page is [unpublished](./jungletv_pages.md#unpublish).

Each JAF application can elect to show a single one of their application pages as a sidebar tab.
If the same application invokes this function with different pages as the argument, the sidebar tab slot available to that application will contain the page passed on the most recent invocation.

#### Syntax

```js
configuration.setSidebarTab(pageID)
configuration.setSidebarTab(pageID, beforeTabID)
```

##### Parameters

- `pageID` - A case-sensitive string representing the ID of the page to use as the content for the tab, as was specified when invoking [publishFile()](./jungletv_pages.md#publishfile).
  When set to `null` or `undefined`, the sidebar tab slot for the JAF application will be removed.
  Connected users with the application's tab active will see an immediate switch to another sidebar tab.
- `beforeTabID` - An optional string that allows for controlling the placement of the new sidebar tab relative to the built-in sidebar tabs.
  The application's tab will appear to the left of the specified built-in tab.
  The built-in tab IDs are: `queue`, `skipandtip`, `chat` and `announcements`.
  If this argument is not specified, the tab will appear to the right of all the built-in tabs.
  The application framework is not designed to let applications control the placement of their tab relative to the tabs of other JAF applications.
  The placement of an application's tab relative to the tabs of other applications may change every time this function is invoked.

##### Return value

In circumstances where the AF runtime is working as expected, this function will return a `true` boolean.

### `setNavigationDestination()`

Sets a new navigation bar item, on the JungleTV client SPA, to link to an application page, registered with [publishFile()](./jungletv_pages.md#publishfile).
The item's label will be the default title passed to [publishFile()](./jungletv_pages.md#publishfile) when publishing the page.

The new navigation destination becomes immediately available on all connected media-consuming clients, and is automatically removed when the application terminates or when the page is [unpublished](./jungletv_pages.md#unpublish).

Each JAF application can elect to show a single navigation destination for one of their application pages.
If the same application invokes this function with different pages as the argument, the navigation bar slot available to that application will link to the page passed on the most recent invocation.

#### Syntax

```js
configuration.setNavigationDestination(undefined)
configuration.setNavigationDestination(pageID, iconClasses)
configuration.setNavigationDestination(pageID, iconClasses, color)
configuration.setNavigationDestination(pageID, iconClasses, color, beforeDestinationID)
```

##### Parameters

- `pageID` - A case-sensitive string representing the ID of the page to use as the destination for the navigation bar item, as was specified when invoking [publishFile()](./jungletv_pages.md#publishfile).
  When set to `null` or `undefined`, the navigation destination for the JAF application will be removed.
- `iconClasses` - A string that defines the icon to show on the navigation bar item, specified as a FontAwesome v5 class list (e.g. `"fas fa-home"`).
  Required only if `pageID` is specified, that is, if the navigation bar item isn't being removed.
  [Free](https://fontawesome.com/v5/search?m=free) and [brand](https://fontawesome.com/v5/search?f=brands) FontAwesome v5 icons can be used.
- `color` - An optional string containing a color for the navigation bar icon.
  The allowed color names are: `"gray"`, `"red"`, `"yellow"`, `"green"`, `"blue"`, `"indigo"`, `"purple"` and `"pink"`.
  If this parameter is not specified or if it is set to `null` or `undefined`, the item will have the color `"gray"`.
- `beforeDestinationID` - An optional string that allows for controlling the placement of the new navigation bar item relative to the built-in destinations.
  The application's destination will appear "before" the specified built-in destination.
  The meaning of "before" depends on the particular destination ID specified and the size of the screen displaying the JungleTV SPA.
  On certain screen sizes, some destination IDs cause the destination to appear directly on the bar while others place it under an overflow menu.
  The built-in destination IDs are: `"enqueue"`, `"rewards"`, `"leaderboards"`, `"about"`, `"faq"`, `"guidelines"` and `"playhistory"`.
  If this argument is not specified, the new item will appear after all the built-in ones.
  The application framework is not designed to let applications control the placement of their navigation destination relative to the ones of other JAF applications.
  The placement of an application's navigation destination relative to those of other applications may change every time this function is invoked.

##### Return value

In circumstances where the AF runtime is working as expected, this function will return a `true` boolean.

### `setUserVIPStatus()`

Requests that a user be considered VIP, or releases that request.
VIP users are able to enqueue media while enqueuing is [restricted to staff or to users who know a password](./jungletv_queue.md#enqueuingpermission), and optionally have special roles associated with them in chat.

Multiple applications may request that the same user be considered VIP, possibly with different appearances.
In such cases, the most recently requested and not-yet-released appearance will be considered, and a user will not cease to be considered VIP until all applications have released their requests regarding that user.
The resulting set of VIP users corresponds to the union of the requests made by all running JAF applications, plus the set of users manually defined on-demand by JungleTV staff.

Applications do not have control over the entire set of VIP users, they can only affect the requests made by themselves.
Requests are automatically released when the application terminates.

#### Syntax

```js
configuration.setUserVIPStatus(address, appearance)
```

##### Parameters

- `address` - The reward address of the user whose VIP status should be affected.
- `appearance` - A string containing one of the following values, defining the special appearance that the VIP user should have, or `undefined` to release this application's request for this user to be considered VIP:
  - `"normal"` - The user will appear as a user with no special privileges, even if they normally have elevated privileges.
  - `"vip"` - The user will appear as a "VIP User" with a crown or similar icon next to their name in chat.
  - `"moderator"` - The user will appear as a "Chat moderator" with the icon associated with chat moderators next to their name in chat.
    They will not actually have any privileges other than the ones usually available to VIP users.
  - `"vipmoderator"` - The user will appear as a "VIP chat moderator" with a specially colored version of the icon associated with chat moderators next to their name in chat.
    They will not actually have any privileges other than the ones usually available to VIP users.
  - `undefined` - Releases this application's request for this user to be considered VIP.
    The user may not immediately lose their VIP status if other running applications are still requesting for this user to be considered VIP.