# Application wallet

Each JungleTV AF application has a [Banano](https://banano.cc/) account associated with it, which the application may use to receive and send Banano.
Among other things, applications may use the balance to set Banano rewards for [application pages enqueued by them](./queue.md#enqueuing-pages).
Within the framework, this Banano account is referred to as the "application wallet" and is primarily interacted with via the [`jungletv:wallet` module](../reference/server/jungletv_wallet.md).

The JAF provides an abstraction layer over the lower-level node interactions needed to send and receive funds.
This layer automatically takes care of waiting for block [confirmations](https://docs.nano.org/glossary/#confirmation), receiving [_receivable_ transactions](https://docs.nano.org/glossary/#receivable) when necessary, ignoring very small "dust" amounts that are often associated with network spam, generating the [computational work](https://docs.nano.org/glossary/#proof-of-work-pow) the Banano network requires to protect itself against spam, signing new blocks with the correct private key and [frontier](https://docs.nano.org/glossary/#frontier), and finally submitting them to the network.
The result is a very simple API through which receiving and sending Banano requires interacting with no more than two functions.

Applications are unable to change the Banano account assigned to them, and do not have access to the private key of this account.

## Working with amounts

Banano amounts are typically presented as a decimal number with up to 29 places of precision after the decimal separator.
Meanwhile, at a technical level, the Banano network and the node RPC commands represent amounts as integer numbers corresponding to the smallest indivisible unit, called _raw_.
This helps avoid mathematical errors associated with accidental conversion to floating-point numbers.
Thus, 1 BAN is represented as `100000000000000000000000000000` raw and 1 raw corresponds to 0.00000000000000000000000000001 BAN.

The JungleTV AF works with amounts represented as raw, too.
However, because in JavaScript the number type uses 64-bit floating-point numbers, it can only accurately represent integers no bigger than 9007199254740991, as explained on e.g. the [relevant MDN page](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/MAX_SAFE_INTEGER#description).
As we just saw, nearly any Banano amount, when represented as raw, is going to correspond to an integer that can't be exactly represented using JavaScript numbers.
Because of this, the JungleTV AF deals with Banano amounts by representing them as strings containing the raw integer, in decimal.
We will generally refer to this format as "raw amount."

Naturally, raw amounts can't be easily manipulated using the regular JavaScript mathematical operators.
To help deal with raw amounts, JAF offers a number of utility functions, including:
- The function [`parseAmount()`](../reference/server/jungletv_wallet.md#parseamount) converts a string containing a Banano amount represented in the human-friendly decimal form, e.g. `"1.023"`, to a string containing its raw form.
- The function [`formatAmount()`](../reference/server/jungletv_wallet.md#formatamount) does the opposite, converting a raw amount, e.g. `"102300000000000000000000000000"`, to a human-friendly value.
- The function [`compareAmounts()`](../reference/server/jungletv_wallet.md#compareamounts) allows for comparing two raw amounts.
- The function [`addAmounts()`](../reference/server/jungletv_wallet.md#addamounts) adds together raw amounts.

Because raw amounts are just strings, application code can naturally include logic to manipulate these amounts as needed.
For example, dividing an amount by 10 can be easily done by removing the last digit of the string.

> **Note**: make sure that floating-point numbers do not end up being accidentally used when dealing with amounts - even setting JavaScript limitations on integer representation aside, it is rarely a good idea to involve floating-point numbers in currency calculations.

## Receiving Banano

For a use case that does not require detecting the transfer of the amount as it happens, an application can receive Banano by simply having it be transferred to its associated Banano account, whose address is accessible via the [`address` property of the `jungletv:wallet` module](../reference/server/jungletv_wallet.md#address).

```js
// Server code

const wallet = require("jungletv:wallet");

console.log("Deposit into the application through the address: " + wallet.address);
```

Almost any amount received on this address will be available for the application to spend very quickly, as soon as the send block is confirmed on the Banano network, which typically happens in less than a second.
"Almost any," because, as a protection against network spam, very small amounts will be considered too small to be worth receiving.
(Small amounts still require significant computational resources in order to be received, as part of the Banano network's spam protection mechanism based on proof-of-work.)
Presently, amounts under 0.001 BAN, or 100000000000000000000000000 raw, will not be received and will not be considered part of its balance.

### Checking the balance

JAF applications can check their Banano balance via the [`getBalance()` function](../reference/server/jungletv_wallet.md#getbalance).

```js
// Server code

const wallet = require("jungletv:wallet");

let balance = await wallet.getBalance();

console.log(`The balance of the application is ${wallet.formatAmount(balance)} BAN`);
```

> **Note**: calling the `getBalance()` function will take care of receiving any receivable transactions.
> Thus, the returned promise may take more or less time to resolve depending on the number of receivable transactions.

### Payment flows

Use cases that require actively waiting for amounts to be received should set up a payment flow.
For example, if an application wants to lock a certain action behind a payment, then it should use payment flows.
This is similar to the process for enqueuing media on JungleTV, where each request is given its own temporary payment address and users have a limited time window to complete the transfer.
Payment flows make it possible for applications to distinguish what is being paid for, even when multiple users are simultaneously paying for different things, because each user will be making transfers to a different address.

The [`receivePayment()` function](../reference/server/jungletv_wallet.md#receivepayment) is used to set up a payment flow.
Behind the scenes, the JAF will temporarily allocate and monitor a Banano account, separate from the one associated with the application.
The returned promise will resolve to a payment flow, from which the application may obtain the temporarily allocated Banano address where payments should be sent.
The application code may then attach event listeners in order to detect when Banano is transferred.
When sufficient funds are transferred or when another condition is met, the payment flow can be closed explicitly, or one may wait for it to expire automatically.
When the payment flow closes, the received funds will be available as part of the application balance.

To help explain how to use `receivePayment()`, the following example launches a payment flow that waits, for up to ten minutes, for at least one BAN to be transferred.

```js
// Server code
const wallet = require("jungletv:wallet");
const oneBAN = wallet.parseAmount("1");

// Wait for up to 10 minutes (10*60*1000 milliseconds)
let paymentReceiver = await wallet.receivePayment(10*60*1000);

paymentReceiver.addEventListener("paymentreceived", (tx) => {
    console.log(`Received ${wallet.formatAmount(tx.amount)} BAN from ${tx.from}`);
    if (wallet.compareAmounts(tx.balance, oneBAN) > 0) {
        // Total received balance in this flow exceeds 1 BAN, we're done
        await paymentReceiver.close();
        console.log(`${tx.balance} sent to the application's wallet!`);
    }
});

paymentReceiver.addEventListener("closed", () => {
    console.log("Payment receiver closed");
    console.log(`Stop sending to ${paymentReceiver.address}!`);
})

console.log(`Send at least 1 BAN to ${paymentReceiver.address}`);
```

The funds are available as part of the application wallet balance after the promise returned by `paymentReceiver.close()` resolves, which is also the moment when the `closed` event of the payment receiver is emitted.

## Sending Banano

Sending Banano from an application's Banano balance is easily achievable using [`send()`](../reference/server/jungletv_wallet.md#send), whose returned promise resolves to the send block hash.
This allows the application keep audit logs, and also to point the user towards the block on a block explorer service, so they can confirm for themselves.
Applications may send to any Banano account.

If the application has insufficient balance, the promise will reject.
When using the async/await syntax, this can be handled via a `try ... catch`.

```js
// Server code
const wallet = require("jungletv:wallet");
const amount = wallet.parseAmount("1.9");
const destination = "ban_1hchsy8diurojzok64ymaaw5cthgwy4wa18r7dcim9wp4nfrz88pyrgcxbdt";

try {
    let blockHash = await wallet.send(destination, amount);
    console.log("Sent on block " + blockHash);
} catch {
    console.log("Sending failed, insufficient funds?");
}
```

`send()` takes an optional third argument, that allows for specifying the representative address to use on the send block.
This makes it possible for applications to participate on certain on-chain data encoding schemes, such as Banano NFTs.

## Spending within JungleTV

When [enqueuing application pages](./queue.md#enqueuing-pages) via the [`enqueuePage()` function](../reference/server/jungletv_queue.md#enqueuepage) of the `jungletv:queue` module, JungleTV AF applications may specify a Banano reward to go along with their entry, by setting the optional parameter `baseReward` to a raw amount.
The specified amount will be debited from the application's balance when enqueuing, and distributed among active spectators after the queue entry is done "playing."
The promise returned by this function will reject if there are insufficient funds.

Additionally, keep in mind that the [`send()` function](../reference/server/jungletv_wallet.md#send) may be used to send funds to JungleTV addresses, too.
Namely, applications may want to send Banano to the crowdfunded transaction ("Skip & Tip") receiving addresses, that can be obtained via the [`crowdfunding.skipping.address`](../reference/server/jungletv_queue.md#skipping) and [`crowdfunding.tipping.address`](../reference/server/jungletv_queue.md#tipping) properties of the `jungletv:queue` module.
By simultaneously [monitoring the queue](./queue.md#monitoring-the-queue), this allows applications to contribute extra rewards towards specific queue entries.

## Usage guidelines

JungleTV AF application developers should avoid needlessly sending high volumes of Banano transactions from application wallets, considering that, due to the spam protection mechanisms of the Banano network, each transaction incurs a computational cost.
While that cost is not directly paid by each application in terms of its performance, it still impacts the infrastructure of JungleTV and its partners.
For most applications, this advice should not affect their design in any major way.
Applications that, by design, seem to spam the network with transactions will naturally not pass the [review process](./review_deployment.md).
Meanwhile, applications that, through the course of their execution, are found to unexpectedly create many blocks on the Banano network, will likely be paused and their developers asked to identify possible solutions.

When using the application wallet to directly send out Banano rewards, developers should keep in mind that the fraud protection measures put in place for regular JungleTV spectator rewards are presently not made available to JAF applications, therefore, applications are more susceptible to e.g. users pretending to have multiple identities ("alt-accounts") than JungleTV itself, unless they implement their own protection measures.
Whenever possible, consider rewarding active spectators (e.g. by increasing the rewards for the currently playing media) rather than rewarding large sets of users directly, as this will harness the protections already in place.

Finally, we advise against using application wallets to store more than the Banano strictly necessary for the application to work as designed:
- JungleTV is not in the business of cryptocurrency custody, and is actually impeded from doing so, due to compliance and conflict-of-interest reasons.
  No guarantees are provided regarding the safety or availability of the funds;
- On each JungleTV environment, all users will staff privileges technically have access to the funds on the wallet of any application.
  While there are auditing mechanisms in place, and while all current and former JungleTV team members have so far proven trustworthy, not only is current performance unfortunately not indicative of the future, this also means that there is a reasonable number of key people who can be unwittingly compromised in an attack.
- Access to an application's balance is possible only as long as the application exists, with the private key of the account being derived from application metadata that ceases to exist when the application is deleted - or if the JungleTV database suffers a misconfiguration, corruption or is outright destroyed.

> **Note**: Exporting an application does not export or otherwise back up its private key/wallet seed.
> It is effectively impossible for application developers to obtain the private key of an application wallet.
>
> An application will have different Banano accounts depending on the environment it runs on.
> Always use the [`address` property](../reference/server/jungletv_wallet.md#address) to get the correct address of the application's wallet.

If you foresee that your application will routinely store large volumes of Banano in its wallet, with "large volume" being defined as roughly how much one would pay for 100 bananas in your local market, please discuss your plans with the JungleTV team prior to continuing development efforts.