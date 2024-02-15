# Queue interaction

The media queue is the central component of JungleTV, directly controlling what plays on the service in each moment.
JungleTV AF applications can monitor the queue as it changes in real time: enqueuing of new entries, reordering and removal of entries, change of the playing media, etc.
Applications may interact with the queue with staff privileges, manipulating it in ways inacessible to regular users, allowing them to e.g. automatically perform moderation tasks.
Applications are also able to enqueue custom content to be played, including interactive content, by enqueuing [application pages](./pages.md).
This gives applications another avenue to expand their user interface.

Additionally, JAF applications can control queue admission criteria, limiting enqueuing to privileged users or users who know a shared secret ("password"), and adjust the multipliers that control enqueue pricing.
They can also control the availability of certain queue features, like the ability for users to reorder or skip queue entries, or make it so that newly added entries are forcibly set to be unskippable at no additional cost - a setting that is used in many popular community events.
Finally, applications are able to control certain aspects of the "Skip & Tip" features (Crowdfunded skipping and Community tipping), which throughout the JAF documentation are typically designated as "crowdfunded transactions."

With this much power, applications - and their developers - also end up having a lot of responsibility.
It is technically possible to build an application that constantly shuffles the media queue, for example - but that doesn't mean it is a good idea.
In order to establish a baseline of what is expected, there is a set of [guidelines](#usage-guidelines) on how applications should interact with the media queue.

## Enqueuing pages

Applications may enqueue one of their published [pages](./pages.md) as if it were any other form of media.
Like regular media, they can be inserted at different points in the queue, being possibly set as unskippable or concealed (i.e. not revealing their title until they begin playing), and once enqueued, they may be removed and moved up or down by other users (depending on queue settings), by staff or by applications.

Once an application page reaches the top of the queue and begins _playing_, it will appear in the JungleTV clients in the same place where the usual media players otherwise would, and users are able to interact with it as they would be able to interact with any other web page.
Then, the [client-server communication](./rpc.md) features can be used to build interactive experiences involving all the spectators.

A published application page may be enqueued using the [`enqueuePage()` function](../reference/server/jungletv_queue.md#enqueuepage).
Application pages may be enqueued with an indefinite media length, meaning that the queue will not advance until either the application or a JungleTV team member explicitly removes it.
Alternatively, and without loss of the ability to remove the entry at any time, an explicit duration between one second and one hour may be specified.

```js
// Server code
const queue = require("jungletv:queue");
const pages = require("jungletv:pages");

pages.publishFile("some-random-id", "page.html", "Example");

// Enqueue the page with a length of 3 minutes.
// It will appear with the default title given to the page, "Example".
let entry = await queue.enqueuePage("some-random-id", "later", 3*60*1000);

// Alternatively, set the page to play after the currently playing entry,
// for an indefinite duration.
// Set it to be unskippable by regular users, and use a custom title.
let entry = await queue.enqueuePage(
    "some-random-id", "aftercurrent", undefined,
    {
        "title": "Custom title",
        "unskippable": true,
    });

// In the second case, the entry will have to be explicitly removed later:
entry.remove();
```

The ability to enqueue application pages is exclusive to applications: neither regular users nor staff are able to manually enqueue them.
Each application is equally unable to enqueue pages published by other applications.

Via the optional parameters to [`enqueuePage()`](../reference/server/jungletv_queue.md#enqueuepage), applications are able to directly associate a Banano reward with the queue entries they create, so that users can be rewarded when the page performance is done, as what happens with the overwhelming majority of queue entries who had a user directly pay for them.
This reward will be debited from the application's [wallet](TODO) when the page is enqueued, and enqueuing will fail if there are insufficient funds.

Unlike what happens when regular users enqueue, applications are not subject to any minimum limits or special behaviors associated with the reward amounts they define for their entries.
We suggest using [`computeEnqueuePricing()`](../reference/server/jungletv_queue.md#computeenqueuepricing) to compute a minimum fair base reward amount that takes into account the current conditions.
Applications may also send rewards directly from their wallet into the [crowdfunded tipping address](../reference/server/jungletv_queue.md#crowdfunded-tipping-status-object) while their queue entries are playing.

Queue entries for application pages are automatically removed from the media queue whenever the respective application pages are unpublished, which also happens whenever the application instance stops for any reason (including its self-termination, staff-initiated termination or due to an overarching server restart).

For more information on application page enqueuing, the available options, and the returned queue entry object, refer to the [`enqueuePage()`](../reference/server/jungletv_queue.md#enqueuepage) reference.

## Monitoring the queue

Applications may access the contents of the queue and monitor the addition, removal and shuffling of queue entries, or just detect when new media begins playing.
Accessing the current contents of the queue is possible via the [`entries` property](../reference/server/jungletv_queue.md#entries), which is an array of [queue entry objects](../reference/server/jungletv_queue.md#queue-entry-object).
To be notified of changes to the queue as they happen, applications may register an event listener, using the [`addEventListener()` function](../reference/server/jungletv_queue.md#addeventlistener) of the `jungletv:queue` module, for one of the relevant [events](../reference/server/jungletv_queue.md#addeventlistener).

For example, to be notified when the currently playing media changes, applications may follow this pattern:

```js
// Server code
const queue = require("jungletv:queue");

queue.addEventListener("mediachanged", (event) => {
    if (event.playingEntry) {
        console.log("New entry just began playing: " + event.playingEntry.media.title);
    } else {
        console.log("The queue just became empty!");
    }
});
```

JungleTV AF applications are also able to access queue settings and adjacent statistics.
For instance, the [`playingSince` property](../reference/server/jungletv_queue.md#playingsince) represents the moment since which media has been playing non-stop on the service, and the [`enqueuingPermission` property](../reference/server/jungletv_queue.md#enqueuingpermission) indicates whether enqueuing is enabled and for which users.

Finally, the `jungletv:queue` module also gives access to the "Skip & Tip" status and controls.
See the [forthcoming relevant section](#crowdfunded-transactions) for more information.

## Manipulating the queue

Applications have moderation privileges over the queue, and therefore are able to manipulate it in even more ways than those accessible to regular users.
Applications may [remove](../reference/server/jungletv_queue.md#removeentry) and [reorder](../reference/server/jungletv_queue.md#moveentry) queue entries, even those not created by them, and even when the self-removal of entries is disallowed.
Users will be refunded for queue entries that are removed before they begin playing.
Unlike regular users, applications do not have to spend [JP](./jp.md) to move entries in the queue, but since applications still have an associated JP balance, developers can choose to play fair by using the [`moveEntryWithCost()` function](../reference/server/jungletv_queue.md#moveentrywithcost) instead of `moveEntry()`.

Because all applications share the same powerful access to the queue, and because it is desirable that the queue behavior remains recognizable and coherent with what is explained to users in the JungleTV client UI and other user help resources, it is important to not go overboard and abide by the queue interaction [guidelines](#usage-guidelines).

### Controlling settings

Being the central component of JungleTV, the queue has a relatively large number of settings that can alter its behavior.
Most of these were introduced to make it possible to host some kinds of special events, or to prevent users from "gaming the system" during certain contests.
For the most part, it is not expected that these settings will remain on their non-default values for indefinite periods of time.

> **Note**: the JungleTV AF does not automatically revert any of the queue settings to their previous values, whenever an application instance is terminated.
> Since the settings may be changed at any moment, by any application and also manually by staff, it would be difficult to decide which would be the correct values to revert to.

One relatively common practice during certain events is to temporarily limit what type of users can enqueue media.
As an example, applications can limit media enqueuing to staff members and users who have been given a password using the [`setEnqueuingPermission()` function](../reference/server/jungletv_queue.md#setenqueuingpermission):

```js
// Server code
const queue = require("jungletv:queue");

queue.setEnqueuingPermission("enabled_password_required", "5uper5ecretp4ssw0rd");

// Later, enqueuing should be reopened to all users, either manually by staff,
// or by the application itself:
queue.setEnqueuingPermission("enabled");
```

Another setting that is commonly used during special events is that of the insert cursor, which controls at which point in the queue new entries are added, when they are not used to skip the currently playing entry and when they are not set to play after the currently playing entry.
The insert cursor appears in the JungleTV clients as a red line in the queue, representing the position where new entries will be added.
The insert cursor can be controlled by setting the [`insertCursor` property](../reference/server/jungletv_queue.md#insertcursor) to the ID of the queue entry _before_ which the cursor should appear.

```js
// Server code
const queue = require("jungletv:queue");

// This code assumes that there are currently more than 3 queue entries,
// and that we want new entries to be placed no later than the entry
// that is presently in the fourth position:
queue.insertCursor = queue.entries[3].id;

// Later, the insert cursor should be cleared:
queue.insertCursor = undefined;
```

The cursor is automatically deactivated when the entry set as the insert cursor begins playing or when it is removed.
For an application to keep the cursor at a constant position, it will need to [monitor for queue changes](#monitoring-the-queue) and keep adjusting `insertCursor` as needed.
To minimize the chances of an unexpected removal of the queue entry that is currently set as the insert cursor, it may make sense to disallow the self-removal of queue entries via the [`removalOfOwnEntriesAllowed` property](../reference/server/jungletv_queue.md#removalofownentriesallowed) - however, keep in mind that both staff and other applications may remove any queue entry, or (re)move the insert cursor, at any time.

JAF applications are able to change the multipliers that control the cost of enqueuing media and of performing crowdfunded skipping, via the [properties of the `pricing` object](../reference/server/jungletv_queue.md#pricing-properties).

For information on all of the controllable queue settings, see the [`jungletv:queue` module reference](../reference/server/jungletv_queue.md#properties).

## Accessing the history

When queue entries are done playing, they are removed from the queue and thus are no longer accessible via the [`entries` property](../reference/server/jungletv_queue.md#entries).
It is possible to obtain information on past media performances, by using the [`getPlayHistory()`](../reference/server/jungletv_queue.md#getplayhistory) and [`getEnqueueHistory()`](../reference/server/jungletv_queue.md#getenqueuehistory) functions.
Both functions operate on the same set of data, but the former returns results ordered how they ended up playing - remember that queue entries may be reordered or inserted at different positions - while the latter returns results sorted by the time at which they were added to the queue.
Entries that were removed from the queue before they began playing are not accessible.

```js
// Server code
const queue = require("jungletv:queue");

// Fetch all the performances that started throughout April 20, 2023:
let oldPerformances = await queue.getPlayHistory(
    new Date("2023-04-20T00:00:00Z"),
    new Date("2023-04-21T00:00:00Z"));

// Fetch the three most recent media performances,
// including the currently playing one, if one exists:
let latestThree = await queue.getPlayHistory(new Date(), new Date(0), {
    "descending": true,
    "includePlaying": true,
    "limit": 3,
});

// Obtain the first ten performances enqueued on December 19, 2023:
let firstTen = await queue.getEnqueueHistory(
    new Date("2023-12-19T00:00:00Z"),
    new Date(),
    {"limit": 10});
```

These functions return an array of objects representing "media performances."
These are partial versions of the queue entry objects, as they can no longer be moved in or removed from the queue, and the information on who may have moved them while in the queue is not stored.
Refer to the [media performance object](../reference/server/jungletv_queue.md#media-performance-object) reference for information on each of the fields of the returned objects.

## Crowdfunded transactions

JungleTV AF applications can monitor the status and control certain aspects of the "Skip & Tip" features, also known as "crowdfunded transactions."
The [`crowdfunding` object](../reference/server/jungletv_queue.md#crowdfunding-object) exposes a number of related methods, properties and events.

For example, applications can monitor when contributions to the skipping and tipping accounts are made, by listening to the [`transactionreceived` event](../reference/server/jungletv_queue.md#transactionreceived) of the `crowdfunding` object:

```js
// Server code
const queue = require("jungletv:queue");
const wallet = require("jungletv:wallet");

queue.crowdfunding.addEventListener("transactionreceived", (event) => {
    if (event.txType === "skip") {
        console.log("New contribution of " + wallet.formatAmount(event.amount) + " BAN to crowdfunded skipping");
    } else if (event.txType === "tip") {
        console.log("New contribution of " + wallet.formatAmount(event.amount) + " BAN to crowdfunded tipping");
    }
});
```

Applications may also control whether crowdfunded skipping is possible by setting the [`crowdfunding.skippingEnabled` property](../reference/server/jungletv_queue.md#skippingenabled), and obtain the current status of the skipping and tipping features via the [`crowdfunding.skipping`](../reference/server/jungletv_queue.md#skipping) and [`crowdfunding.tipping`](../reference/server/jungletv_queue.md#tipping) properties, respectively.

## Usage guidelines

JAF applications have as much or even more power over the media queue as JungleTV staff, augmented by similarly high levels of control over other JungleTV aspects and the fact that, being computer programs, they can react much more quickly and consistently to different events.
This means that, in theory, applications can greatly change the usual mechanics of the queue, to the point where it barely looks like a "queue" and no longer fits the descriptions given on the JungleTV client SPA and other JungleTV documentation, past user testimonies, etc.

Just like a "power tripping" staff member would be called out and eventually demoted, applications too are discouraged from going too overboard with their abilities.
Special queue behaviors are permitted during special events, but it is important to not disappoint users accustomed to the usual queue behavior and thus the typical "feel" of JungleTV.
Such special behaviors should be clearly announced, e.g. as part of an event announcement, and limited to specific time frames, with the normal behaviors being reestablished once the event is over.

Naturally, what is acceptable for an application to do depends on its goals.
Consider an application intended to automatically moderate the queue, by e.g. processing the titles of queue entries, in order to automatically identify and perhaps even remove content that doesn't abide by the JungleTV guidelines.
Such an application will likely be designed to run at all times, changing the baseline of what is considered the normal behavior of the JungleTV platform, and this is not a behavior that necessarily needs to be announced or to be limited to a specific period of time.
Meanwhile, an application that checks whether all entries being enqueued match a specific theme, and removes those which don't, is atypical behavior that should be pre-announced and be limited in time.

Finally, applications should not end up giving users more power over the queue than they usually have, in an indiscriminate fashion.
For example, while it is technically possible and even quite simple to make an application that allows any user to reorder queue entries without costing them [JP](./jp.md) and without any other restrictions, such an application is unlikely to pass the [review process](./review_deployment.md).
The restrictions, costs and rate-limits associated with certain actions were carefully thought out, and put in place in an effort to avoid making JungleTV too chaotic.
Imagine that the order of queue entries was constantly being shuffled by arbitrary users, or even automatically by an application: those who spent more Banano to enqueue entries closer to the top of the queue could feel that their contribution was wasted.

Applications giving users new ways to access features that are usually restricted to regular users, must consider associating some cost or other limitation with their actions.
For example, an application that allows users to vote for the removal of a queue entry, should make it so that voting costs some JP or Banano, in order to prevent such actions from being performed too often by users (perhaps even automatically, in a fraudulent fashion).

### Guidelines for enqueued content

The content enqueued by applications, via enqueued application pages, is generally subject to the same community guidelines as the rest of the content submitted for reproduction on JungleTV.
This means that disturbing, shocking, adult/mature or excessively repetitive content, unauthorized advertising and all other forms of unauthorized content should equally not be enqueued by applications.

Applications that allow user-generated/user-selected portions to appear as part of their enqueued application pages should, if possible, try to automatically prevent such unauthorized content from being included.
For example, consider an application that allows users to pick images from a third-party service, that are then affixed in a virtual board, in order to build a collaborative art piece of sorts.
The application should exclude images whose metadata on the service indicates that they are not safe for work, or even configure search parameters in order to automatically exclude content tagged as such.

Ultimately, it is up to manual moderation efforts to identify potentially infringing content and act accordingly.
Therefore, applications should be prepared for their enqueued content to be removed at any point, before or after its performance on the service begins.
To avoid a complete removal of the queue entry, applications may offer specialized moderation features that allow for the removal of specific content portions.
Make sure such features are documented as part of the [submission process](./review_deployment.md#submitting-an-application).
Should an application end up requiring too much moderator intervention, the JungleTV team reserves the right to cease offering it on the service, as explained in the [post-deployment rights and duties](./review_deployment.md#post-deployment-rights-and-duties) section.

### Reward guidelines

Applications are able to enqueue application pages at no cost.
If no community tipping contributions are made during their performance, such queue entries may end up playing without paying any Banano reward to active spectators.
While this does not cause any problems from a technical point of view, this goes against what users expect of the platform.

As a general rule of thumb, applications should avoid having JungleTV play content without an associated reward, for more than one consecutive minute.
This may be quite difficult to accurately enforce, considering that different applications and even JungleTV staff may be enqueuing entries without an associated reward.
Therefore, in general, applications should always try to have some reward associated with their queue entries, even if minimal - particularly, if their content is playing at times when different, paying content could be playing.
This balance naturally changes in the case of content that is enqueued as filler, during the periods when nothing would otherwise be playing.

[`computeEnqueuePricing()`](../reference/server/jungletv_queue.md#computeenqueuepricing) can be used to compute a reward amount that would be in line with what a regular user would need to pay, in order to enqueue content of a given length on the service.
Applications can also dynamically adjust the rewards for the currently playing queue entry by sending funds from their [wallet](TODO) to the [crowdfunded tipping address](../reference/server/jungletv_queue.md#tipping).