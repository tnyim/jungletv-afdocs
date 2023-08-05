# JP interaction

Banano (BAN) and JungleTV Points (JP) are the two fungible currency-likes associated with JungleTV.
While Banano can be mistaken for actual money, Points are an in-website balance which cannot be redeemed for anything resembling real money nor directly transferred between users.
JP are very similar to the in-game currencies found in many multiplayer games, and are used to create a secondary economy within JungleTV, which is mostly independent from Banano.

JP are awarded to users for performing certain actions within the website, such as enqueuing media or participating in chat.
Generally, users are rewarded in Points whenever it would be impractical to reward them with Banano.
This is because JP do not have a fixed supply - JungleTV can create new Points at any time - while Banano is scarce due to having a fixed supply, meaning that Banano rewards typically come from funds sent by community members or - less frequently - the Banano distribution fund.
Points also makes it possible to more visibly reward users for actions that require them to spend Banano - because rewarding users with BAN for spending BAN would simply be a discount.
It is also possible to directly spend Banano to receive JP, but not the other way around.

Points can be spent exclusively within JungleTV; they are used to artificially increase the cost of certain actions within the website.
These are typically actions which, if available at zero cost, could be performed repeatedly indefinitely, to annoy other users or otherwise introduce too much chaos in the JungleTV operation.
Examples include sending GIFs in chat or reordering queue entries.

When deducted in large quantities, Points can also be used to assert the age or "weight" of an account and, in a way, help deduplicate identities.
To amass large quantities of Points, users need to either participate in JungleTV for an extended period of time, or spend a non-negligible amount of Banano.
This becomes impractical for a ill-intentioned user intending to operate multiple identities simultaneously.
JungleTV gates certain privileges, such as reduced activity check (CAPTCHA) frequency, behind its JungleTV Nice subscription, which must be bought every month for a large sum of JP.

The revenue originating from purchases of JungleTV Points with Banano is used to complement the "rounding error fees" captured with each played media entry, and is used to reward JungleTV moderators and developers.

## Awarding and deducting Points

JungleTV AF applications can manipulate a user's JP balance, using the [`createTransaction()` function](../reference/server/jungletv_points.md#createtransaction) of the `jungletv:points` module to create JP transactions.
The value of a transaction must be a non-zero integer.
Transactions with negative values deduct points from the user's balance, and positive values increase the user's balance.
The JP balance of an user cannot be negative - attempts to create transactions which would bring a balance below zero result in an [exception](../reference/server/jungletv_points.md#exceptions).

Transactions created by JAF applications must include a description which is user-facing, displayed in their JP transaction history.
The ID and version of the JAF application are registered along with the transaction, for auditing purposes.

```js
// Server code
const points = require("jungletv:points");

points.createTransaction(
    "ban_1tx9zgxaebdimrcgsguk8hcobfmry11wyetkybhh3pmwbtocs5cihm88jrw6",
    "Example that awards 19 points", 19);
```

## Checking Points balance

JAF applications can check the points balance of a user using the [`getBalance()` function](../reference/server/jungletv_points.md#getbalance).

```js
// Server code
const points = require("jungletv:points");

const balance = points.getBalance(
    "ban_1tx9zgxaebdimrcgsguk8hcobfmry11wyetkybhh3pmwbtocs5cihm88jrw6");
```

> **Note**: while this can be used to check whether a user has sufficient points to perform some action before attempting to deduct them, simply attempting to deduct them and handling the exception that may occur is a sufficient and preferable approach, that is free of possible concurrency issues, such as time-of-check to time-of-use bugs.

`getBalance()` can be used, for example, to show certain application features as unavailable in the application UI, when the user has insufficient points to use them.
In such cases, it is good practice to point users towards the page where they can acquire JP with Banano:

```js
// Client code

// Some button/link in the application page
const button = document.getElementById("purchase-points-button");

button.addEventListener("click",
    (event) => window.appbridge.navigate("/points/frombanano"));
```

## Checking JungleTV Nice subscription status

Applications can check whether the user is subscribed to JungleTV Nice, and until when, using the [`getNiceSubscription()` function](../reference/server/jungletv_points.md#getnicesubscription).

```js
// Server code
const points = require("jungletv:points");

const subscription = points.getNiceSubscription(
    "ban_1tx9zgxaebdimrcgsguk8hcobfmry11wyetkybhh3pmwbtocs5cihm88jrw6");

if (subscription) {
    console.log(`User is subscribed to JungleTV Nice since ${subscription.startsAt}`);
} else {
    console.log("User is not subscribed to JungleTV Nice");
}
```

Subscriptions may last for an indefinite period, with the subscription's `endsAt` field changing with every consecutive renewal.
The number of consecutive renewals is easy to count through the length of the subscription's `paymentTransactions` array.
If a user lets their subscription lapse and then subscribes again, that is considered a new subscription, with a different `startsAt` and a new set of `paymentTransactions`.

## JP economy guidelines

JungleTV AF applications are encouraged to make use of the JungleTV Points system, both to associate a "light" cost with certain actions, and to reward users for other actions.
However, the use of the JP system must be carefully planned, as to not imbalance the JP economy.

While it is definitely technically possible to develop an application that gives ten million JP to every user, such venture would immediately make the points system redundant, as users would be able to spend points without consequence on actions that should be "expensive;" simultaneously, most existing JP rewards would become meaningless.

Application developers should turn to JungleTV staff for guidance on what value and frequency of JP rewards are most appropriate for their applications.
The end goal should be to avoid an ever-increasing "inflation" in the JP economy, a scenario that is otherwise likely: if left unchecked, different applications could begin trying to one-up each other regarding their JP rewards.
JP transaction values are subject to adjustment as part of the [review and deployment](./review_deployment.md) process.

Applications are also encouraged to reduce the JP cost of certain actions to users who are subscribed to JungleTV Nice, when it makes sense.
The high JP cost of Nice subscriptions is meant to be offset by discounts in actions that require JP, but not all actions need to be discounted for Nice subscribers.

Additionally, one should also take into account that each JP transaction has a small computational and storage cost associated, as well as a user experience impact, in the sense that too frequent transactions will fill the JP transaction log, making it harder for users to analyse it.
The individual per-transaction costs are very small, but can add up quickly.
Therefore, creating transactions too frequently on a per-user basis is to be avoided.
As an example, a "hidden object" game might create a JP transaction for every object a user finds.
However, if there are dozens of objects per level, this could result in dozens of transactions being created every minute, perhaps times dozens of users.
In such case, it makes more sense to add up the points within each level, and create a single transaction at the end of every level or round.
