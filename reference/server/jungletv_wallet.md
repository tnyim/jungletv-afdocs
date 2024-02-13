# `jungletv:wallet` module

The `jungletv:wallet` module allows for interaction with the application's own Banano account.

This module is not imported by default. To use this module, import it in your server scripts as follows:

```js
const wallet = require("jungletv:wallet")
```

## Methods

### `getBalance()`

Obtains the usable balance of the Banano account that is associated with this application.
This is a sum of all the received balance and all non-dust receivable balance.

#### Syntax

```js
wallet.getBalance()
```

##### Parameters

None.

##### Return value

A promise that will resolve to the raw Banano amount effectively usable by the application.

### `send()`

Sends a Banano amount from the account associated with this application.

#### Syntax

```js
wallet.send(address, amount)
wallet.send(address, amount, representative)
```

##### Parameters

- `address` - The destination address to which to send Banano.
- `amount` - A string containing the amount of Banano to send, in raw units, which will be debited from the balance of the application account.
- `representative` - An optional string, containing the representative address to use on the send block.
  Can be used to include extra information as part of on-chain encoding schemes (e.g. Banano NFTs).

##### Return value

A promise that will resolve to the hash of the transaction send block.

### `receivePayment()`

Launches a new payment flow, allowing the application to receive Banano until a specific amount is received or other condition is met.
This temporarily allocates a separate Banano account, into which users should send Banano, and issues [events](#payment-receiver-events) whenever it receives new non-dust transactions.
This allows for building payment flows similar to those used for media enqueuing by JungleTV.

#### Syntax

```js
wallet.receivePayment(timeout)
```

##### Parameters

- `timeout` - A number representing the duration, in milliseconds, for which to wait for a payment, after which the flow will close and any amount received will be sent to the application's [`address`](#address).
  Must be no shorter than 30 seconds and no longer than 10 minutes.

##### Return value

A promise that will resolve to a [payment receiver](#payment-receiver-object) object that can be used to monitor and prematurely close the payment flow.

### `compareAmounts()`

Utility function that compares two raw Banano amounts.

#### Syntax

```js
wallet.compareAmounts(first, second)
```

##### Parameters

- `first` - A string containing the first amount to compare, in raw Banano units.
- `second` - A string containing the second amount to compare, in raw Banano units.

##### Return value

A number that is -1 if the first amount is less than the second, 1 if the first amount is greater than the second, and 0 if the amounts are equal.

### `formatAmount()`

Utility function that converts a raw Banano amount into a string containing its decimal representation, using a dot as decimal separator.
For example, the raw amount "123000000000000000000000000000" will be converted to "1.23".

#### Syntax

```js
wallet.formatAmount(amount)
```

##### Parameters

- `amount` - A string containing the amount to format, in raw Banano units.

##### Return value

A string containing the formatted amount.

### `parseAmount()`

Utility function that converts a Banano amount, encoded as decimal with an optional dot as decimal separator (e.g. "123", "1.23"), to a raw amount.
For example, the formatted amount "1.23" will be converted to the raw amount "123000000000000000000000000000".

#### Syntax

```js
wallet.parseAmount(decimalAmount)
```

##### Parameters

- `decimalAmount` - A string containing the amount to parse.

##### Return value

A string containing the parsed amount, in raw Banano units.

### `addAmounts()`

Utility function that adds together multiple raw amounts.

#### Syntax

```js
wallet.addAmounts(amount1, /* ..., */ amountN)
```

##### Parameters

An arbitrary number of strings containing the amounts to sum, in raw Banano units.

##### Return value

A string containing the sum of the specified amounts, in raw Banano units.

### `negateAmount()`

Utility function that calculates the negative of a raw amount.

#### Syntax

```js
wallet.negateAmount(amount)
```

##### Parameters

- `amount` - A string containing the amount to negate, in raw Banano units.

##### Return value

A string containing the specified amount, with its sign flipped, in raw Banano units.

## Properties

### `address`

This read-only string property corresponds to the address of the Banano account associated with this application.

#### Syntax

```js
wallet.address
```

## Associated types

### Payment receiver object

Represents a payment flow that was initiated by [receivePayment()](#receivepayment).

| Field                   | Type     | Description                                         |
| ----------------------- | -------- | --------------------------------------------------- |
| `addEventListener()`    | function | See [`addEventListener()`](#addeventlistener)       |
| `removeEventListener()` | function | See [`removeEventListener()`](#removeeventlistener) |
| `close()`               | function | See [`close()`](#close)                             |
| `address`               | string   | See [`address`](#address-1)                         |
| `balance`               | string   | See [`balance`](#balance)                           |
| `closed`                | boolean  | See [`closed`](#closed)                             |

## Payment receiver methods

### `addEventListener()`

Registers a function to be called whenever the specified [event](#payment-receiver-events) occurs.
Depending on the event, the function may be invoked with arguments containing information about the event.
Refer to the documentation about each [event type](#payment-receiver-events) for details.

#### Syntax

```js
let paymentReceiver = await wallet.receivePayment(/* ... */);
/* ... */
paymentReceiver.addEventListener(type, listener)
```

##### Parameters

- `type` - A case-sensitive string representing the [event type](#payment-receiver-events) to listen for.
- `listener` - A function that will be called when an event of the specified type occurs.

##### Return value

None.

### `removeEventListener()`

Ceases calling a function previously registered with [`addEventListener()`](#addeventlistener) whenever the specified [event](#payment-receiver-events) occurs.

#### Syntax

```js
let paymentReceiver = await wallet.receivePayment(/* ... */);
/* ... */
paymentReceiver.removeEventListener(type, listener)
```

##### Parameters

- `type` - A case-sensitive string corresponding to the [event type](#payment-receiver-events) from which to unsubscribe.
- `listener` - The function previously passed to [`addEventListener()`](#addeventlistener), that should no longer be called whenever an event of the given `type` occurs.

##### Return value

None.

### `close()`

Prematurely closes the payment flow, sending any received amount to the application account.

#### Syntax

```js
let paymentReceiver = await wallet.receivePayment(/* ... */);
/* ... */
paymentReceiver.close()
```

##### Parameters

None.

##### Return value

A promise that resolves when the funds have been sent to the application account.

## Payment receiver properties

### `address`

This read-only string property contains the address of the Banano account into which payments for this flow should be sent.
After a payment flow is closed, this account should not be used to receive further funds.

#### Syntax

```js
let paymentReceiver = await wallet.receivePayment(/* ... */);
/* ... */
paymentReceiver.address
```

### `balance`

This read-only string property represents the amount, in raw Banano units, received in this payment flow so far.
To monitor this amount, use [`addEventListener()`](#addeventlistener) to attach a listener to the [`paymentreceived` event](#paymentreceived).

#### Syntax

```js
let paymentReceiver = await wallet.receivePayment(/* ... */);
/* ... */
paymentReceiver.balance
```

### `closed`

This read-only boolean property indicates whether this payment flow has closed, either by being prematurely closed via [`close()`](#close), or by reaching the timeout specified in the call to [`receivePayment()`](#receivepayment).
After a payment flow is closed, its [`address`](#address-1) should not be used to receive further funds.
To monitor this amount, use [`addEventListener()`](#addeventlistener) to attach a listener to the [`closed` event](#closed-1).

#### Syntax

```js
let paymentReceiver = await wallet.receivePayment(/* ... */);
/* ... */
paymentReceiver.closed
```

## Payment receiver events

Listen to these events using [`addEventListener()`](#addeventlistener).

### `paymentreceived`

This event is fired when a new transaction is received in a payment flow.

#### Syntax

```js
let paymentReceiver = await wallet.receivePayment(/* ... */);
/* ... */
paymentReceiver.addEventListener("paymentreceived", (event) => {})
```

#### Event properties

| Field       | Type   | Description                                                                     |
| ----------- | ------ | ------------------------------------------------------------------------------- |
| `type`      | string | Guaranteed to be `paymentreceived`.                                             |
| `amount`    | string | The amount, in raw Banano units, that was received in this transaction.         |
| `from`      | string | The Banano account address of the sender of the transaction.                    |
| `blockHash` | string | The block hash of the sending transaction.                                      |
| `balance`   | string | The amount, in raw Banano units, that was received in this payment flow so far. |

### `closed`

This event is fired when the payment flow stops being monitored, either because it was explicitly closed via [`close()`](#close) or because it reached the timeout specified in the call to [`receivePayment()`](#receivepayment).

#### Syntax

```js
let paymentReceiver = await wallet.receivePayment(/* ... */);
/* ... */
paymentReceiver.addEventListener("closed", (event) => {})
```

#### Event properties

| Field    | Type   | Description                |
| -------- | ------ | -------------------------- |
| `closed` | string | Guaranteed to be `closed`. |