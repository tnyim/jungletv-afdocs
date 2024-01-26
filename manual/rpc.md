# Client-server communication (RPC)

[Application pages](./pages.md) allow JungleTV AF applications to present a graphical interface, but they would be of limited use if pages could not communicate with the application logic running on the JungleTV server.
This manual section documents how client-side page logic and server-side application logic can communicate, and the performance and security aspects involved in this communication.

[Remote procedure calls](https://en.wikipedia.org/wiki/Remote_procedure_call) (RPC) are a pattern for communication between different machines or execution contexts, in the context of a distributed system.
JungleTV, and JAF applications, are precisely that, a distributed system that uses the client-server model: as mentioned in the [architecture](./architecture.md) section, there is one machine - the JungleTV server - that must communicate with hundreds of other machines - the JungleTV clients.

In line with what's expected when the term RPC is involved, the RPC system in the JungleTV AF takes care of abstracting away the communication between an application's server code and the code running on application pages, on the clients.
Application developers can write code in their application page scripts, that invokes methods implemented on the server scripts, and returns their result - all without having to worry about how the communication between the two sides happens.

There is also an events system that allows application pages to inform the server logic about any happenings of interest, and vice-versa.
In the JAF, the fundamental difference between remote method calls and events is that the former always consists on the client invoking a method on the server, which can return a result to the client; meanwhile, events can be sent from the server to the clients or from a client to the server, but they do not have a result and the sender is not informed about their delivery.
Furthermore, servers are not required to register handlers for all events emitted by their application pages, and pages are not required to handle all events emitted by the server.

> **Aside**: While application developers should not have to worry about it, a simplified representation of the components involved in the communication can be found in the [JungleTV AF architecture diagram](./architecture.md#the-application-framework); specifically, the connections shown in orange represent the path taken by remote method invocations and event data.

## Remote methods

In the context of the JungleTV AF, remote methods are code that exists and executes as part of the server-side logic of an application, and which can be executed on demand by the scripts running within application pages.

Using the [`registerMethod()` function](../reference/server/jungletv_rpc.md#registermethod), server scripts can make methods available to client scripts, under names chosen by the programmer.
Client-side scripts then can invoke those methods using the chosen names, through the [`serverMethod()` appbridge function](../reference/appbridge/api.md#servermethod).
From now on, we will refer to the functions passed to `registerMethod()` as "method handlers."

```js
// Server code
const rpc = require("jungletv:rpc");

function myHandler(context, name) {
  return `Hello ${name}!`;
}

rpc.registerMethod("hello", "unauthenticated", myHandler);
```

```js
// Client code
const name = await window.appbridge.prompt("What's your name?");
const result = window.appbridge.serverMethod("hello", name);
await window.appbridge.alert(result);
```

In the example above, the `myHandler` function (the method handler) will execute whenever a client script calls the remote method with the name `hello`.
`name` is an argument that the client script is expected to send, and the string returned by the method handler, "Hello \[name]!", will be sent back to the client.

When invoking a remote method, arguments are automatically marshalled to JSON on the client and unmarshalled on the server.
An example of something that won't work as expected, is passing a function as an argument to a remote method, because functions are not JSON-serializable.
The full list of JSON serialization restrictions can be found on [the MDN web docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify#description).

Method handlers are invoked with a [context object](../reference/server/jungletv_rpc.md#context-object) as the first argument, as seen in the example above, followed by the deserialized representations of any arguments sent by the client.
It can be used to obtain trusted information about the authenticated user (if any), and also information about which page ID the event originated from (which is reported by the clients and therefore vulnerable to tampering).

Remote method invocations are necessarily asynchronous, i.e. on the client side, the `serverMethod()` function always returns a [`Promise`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises).
The promise resolves to the return value of the function passed to `registerMethod()`, which is eventually `undefined` if the handler returns no value, or an error may be thrown if the invocation fails.
Like function arguments, return values are subject to JSON serialization restrictions.

Method handlers may be synchronous or return promises, depending on their needs.
On the client side, `serverMethod()` returns a promise regardless, and therefore invocations of remote methods can be awaited (e.g. with the `await` keyword, or by using the `then()`-`catch()`-`finally()` promise API).
Scripts on the client side can handle remote method invocation errors, potentially caused by server-side throws or by connection problems, using e.g. a `try` ... `catch` block.

The JAF has an authentication and authorization mechanism built into the remote methods system:
- The [context object](../reference/server/jungletv_rpc.md#context-object), passed as the first argument to method handlers, includes information about the `sender`, including their reward address and permission level.
  The sender is the [User](../reference/server/common_types.md#user-object) signed in to the JungleTV client where the remote call originated from.
  It will be undefined if the call originated from an unauthenticated user.
  The `sender` field, like the rest of the context object, can't be manipulated by the clients.
  Applications can trust the `sender` information to be correct and untampered, without the need to perform any additional checks.
- The [`registerMethod()` function](../reference/server/jungletv_rpc.md#registermethod) takes an argument specifying the minimum permission level necessary to invoke the method being registered.
  Server scripts won't be notified about users attempting to call methods for which they don't have sufficient permissions.

The minimum permission level restriction is effectively equivalent to having the server script check the information in the context object, and throwing when the requirements are not met.
While the notion of specifying a minimum permission level for every remote method may therefore seem redundant, the intent is to encourage application developers to actively think about who should be allowed to perform each possible action on their application.
Besides, this also simplifies application code slightly for the most common use cases of authorization checks, such as making a remote method exclusive for JungleTV staff.
In any case, application developers are free to allow methods to be called by anyone, and implement their own custom authorization logic within their method handlers - for example, to allow methods to be called exclusively by specific users, regardless of permission level.

## Events

[Events](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Building_blocks/Events) are used in the [event-driven programming paradigm](https://en.wikipedia.org/wiki/Event-driven_programming) to represent an occurrence which may be handled within the context of a software system, often in a component that is not the one where the event originated.
Events are used by many of the JavaScript APIs available to scripts running within the browser; among other things, they are used to represent the users' interaction with the different elements of web pages.
The JungleTV AF makes it possible for applications to use events to communicate between the application pages and the application server logic, in both directions.

Using the [`emitToAll()` function](../reference/server/jungletv_rpc.md#emittoall), server scripts can emit events to all of the application's connected pages.
Events can target a specific page and/or user by using the similar functions [`emitToPage()`](../reference/server/jungletv_rpc.md#emittopage), [`emitToUser()`](../reference/server/jungletv_rpc.md#emittouser) and [`emitToPageUser()`](../reference/server/jungletv_rpc.md#emittopageuser).
To listen for events coming from the server, application page script should register event handlers on the [`server` appbridge object](../reference/appbridge/api.md#server).

Events can have arguments, which are marshalled and unmarshalled by the framework, subject to the same restrictions as remote method arguments.
However, unlike remote methods, events do not have the concept of a return value, and there is no built-in method to wait for their delivery.

Let's imagine your server logic keeps a counter, and wants to inform the application pages whenever it is incremented, so it can be fresh on the users' browsers.
This can be done by emitting an event called, for example, `counterUpdated`, from the server to the pages.
The relevant code would look like this:

```js
// Server code
const rpc = require("jungletv:rpc");

counter++;
rpc.emitToAll("counterUpdated", counter);
```

```js
// Client code
window.appbridge.server.addEventListener("counterUpdated", (eventFromServer) => {
    console.log(`Counter is now ${eventFromServer.detail[0]}`);
});
```

This shows how to emit events from the server to the client.
On the client event handler, the arguments passed from the server script end up in the event's `detail` field - as is customary for DOM-originated events.
Because there can be multiple arguments, `detail` is always an array.

Emitting events in the opposite direction, from client scripts to the server, is also possible, using the [`emitToServer()` appbridge function](../reference/appbridge/api.md#emittoserver).
To listen for events coming from application pages, server scripts should use the [`addEventListener()` function](../reference/server/jungletv_rpc.md#addeventlistener) of the `jungletv:rpc` module.
This works similarly to the `registerMethod()` function of remote methods, with the difference that multiple handlers can be registered for the same event.

Let's imagine that now each user keeps an independent counter on their side, and the server should be informed whenever a user updates their counter:

```js
// Server code
const rpc = require("jungletv:rpc");

function counterUpdatedHandler(context, counter) {
  if (context.sender) {
    console.log(`${context.sender.address} updated their counter to ${counter}.`);
  }
}

rpc.addEventListener("userUpdatedCounter", counterUpdatedHandler);
```

```js
// Client code
counter++;
window.appbridge.emitToServer("userUpdatedCounter", counter);
```

As with remote methods, event handlers on server scripts receive the arguments passed by the client, preceded by a [context object](../reference/server/jungletv_rpc.md#context-object) which can be used to obtain trusted information about the authenticated user (if any), and also untrusted information about which page ID the event originated from.

The [context object](../reference/server/jungletv_rpc.md#context-object) additionally contains a `trusted` field which is `true` whenever an event is emitted by the JungleTV AF itself, namely, when an application page instance connects to, or disconnects from, the server.
As usual, the context object allows for discovering the relevant page ID and user.
These events are documented in the [`jungletv:rpc` module reference](../reference/server/jungletv_rpc.md#events).

On both the server and client sides, event handlers can be synchronous or asynchronous - their return value is ignored and execution does not wait for event delivery.

## Performance considerations

The JavaScript execution model is fundamentally single-threaded, so it makes sense to consider how events and remote method invocations, potentially coming from hundreds of clients in virtually the same instant, will be handled by the JAF and by your application code.
Concurrent incoming invocations and events are queued, and their handlers will execute serially within the JavaScript event loop of a JAF application.
As an example, if a method handler is synchronous, takes approximately 1 second to finish its computation, and 100 clients invoke it at virtually the same time, then some clients will wait up to 100 seconds to receive the response (or, most likely, they will see a timeout exception being thrown).

It is therefore important to ensure that method handlers return reasonably quickly:
- If your method handler takes a significant amount of time because it waits for asynchronous processes, then you should make your method handler return a [`Promise`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises), for example, by marking it as `async`.
  This will allow you application's event loop to proceed handling other incoming remote method invocations, events and any other asynchronous tasks that may have been added to the event loop, for example, `setInterval()` handlers.
  This will help keep your application responsive, especially if not all handlers take a significant amount of time, because this way a longer-running handler won't necessarily block the execution of other faster ones.
  However, if the process takes so long as to cause timeouts in the communication, you should use the next approach.
- If your method handler takes a significant amount of time because it needs to perform some computationally heavy task, or simply a task that for whatever reason depends on the passing of actual ("wall clock") time, consider changing the architecture of your request.
  For example, you can use a method invocation to kick off the computation in an asynchronous fashion, and later emit an event to the client containing its result.

The actual meaning of "significant amount of time" will vary depending on the needs of your application.
If it is, for example, a game that must process its main loop multiple times per second, it will certainly be more sensitive to slow handlers than an application which mainly sits idle waiting for a single such heavy request.
When it comes to communication timeouts, relevant when awaiting the execution of remote methods, a good rule of thumb in the JAF is that any task prone to taking more than 5 seconds should be changed to be more asynchronous, for example and as suggested above, by splitting request and response into two different communications.

## Security considerations

Distributed systems, such as JungleTV and JAF applications, are particularly hard to perfect, from an [information security](https://en.wikipedia.org/wiki/Information_security) standpoint.
This is largely due to the complexity that is inherent to them: they have multiple components executing on different machines, which communicate over some medium (the internet, in our case); the various parts are not necessarily developed in tandem which means there may be mismatched expectations between them; and particularly when it comes to web services available to the general public, of which JungleTV and its applications are an example, there always exist countless threats, from the automated and generic kind, to the tailored and targeted kind.
The communication between the various parts of a distributed system - the JungleTV client SPA and server, in our context - is often the attack vector, for example by forging messages between them, impersonating other components or the users of these components in order to perform unauthorized actions or otherwise get the system into a state the developers did not foresee.

The JungleTV Application Framework abstracts away the communication between the application page logic and the server-side logic, enforces or encourages some information security best practices, and takes care of authenticating users and providing the means necessary for applications to authorize their actions as adequate.
However, for applications to be reasonably secure, application developers still need to keep basic security principles in mind, and correctly use the tools at their disposal, to avoid creating vulnerabilities.
This is important, because an insecure application may end up jeopardizing the security of JungleTV itself - as briefly discussed in the [No True Sandboxing section](./architecture.md#no-true-sandboxing).

Let's begin by expanding upon what aspects are handled by the framework, and thus application developers are not necessarily forced to think about:

- The communication between the application pages and the JungleTV server happens exclusively over HTTPS: this means that application developers should not need to worry about attackers intercepting or modifying sensitive information from legitimate requests on their way to the server - with rare exceptions, such as corporate or government-level proxies, which are beyond the control of most web services. (Techniques like certificate pinning can help, but may also render the service unavailable to users who can't avoid such interception.)
  Still, this does not ensure that all requests are _legitimate_, as will be explained later.
- User authentication is handled by JungleTV: server-side application code can trust that whenever user information is present in a [context object](../reference/server/jungletv_rpc.md#context-object), that user is successfully authenticated from JungleTV's point of view.
  For unprivileged users, JungleTV uses long-lived [JSON Web Tokens (JWTs)](https://jwt.io/) for authentication.
  For privileged users, the JWTs are shorter-lived.
  In both cases, the JWTs are automatically renewed when used within their lifetime.
  As in many websites, users are vulnerable to having their session token stolen, but they can mitigate this by cancelling all of their previously issued tokens, at any time.
- User authorization is partially handled by the JAF: information on the permission level of users is included in the context object, and applications can restrict some [remote methods](#remote-methods) to users at or above a certain permission level.
  Application code should implement whatever other authorization measures are deemed necessary in the context of an application.
- The [global rate limiter](#global-rate-limiter), which is detailed later on, enforces a generous maximum limit on how quickly each client can send requests for ingestion by the JAF, which helps prevent a denial of service attack through resource starvation - but application developers should still implement additional rate limiting for specific requests within their application, whenever it makes sense.

### On message _legitimacy_

There are many aspects which the JAF cannot reasonably check because they are application-specific, or because they are an inherent consequence of how a web service operates.

The JungleTV AF cannot ensure that a message (remote method invocation or received event) is _legitimate_ from the point of view of an application developer's intent. This seems like a vague sentence, because the concept is vague, and best explained with an example:

<ul>
  Imagine your application keeps a counter and users can press a button on an application page to increment that counter.
</ul>

The framework cannot reasonably ensure, in a "cheap" way (computationally and user experience-wise), that whenever the application receives a remote method invocation to increment the counter, that it actually results from a human pressing the button on a common web browser, vs. e.g. an automation sending a request that looks exactly like one a browser would send, possibly even using legitimately obtained user credentials.
For this sort of legitimacy check, one would have to implement methods like [CAPTCHAs](https://en.wikipedia.org/wiki/CAPTCHA) and browser fingerprinting, which have their own cost-benefit balance on multiple fronts: application complexity, computational requirements, user experience drawbacks... only the application developer would know which checks are most appropriate or whether no checks are actually needed.

_Illegitimate_ requests can exploit a lack of verification of method invocations and their arguments:

<ul>
  Consider the previous example, but each user is allowed to increment the counter just once.
  To keep users from pressing the counter more than once, you disable the button on the page once it has been pressed.
</ul>

If the application's server logic does not keep track of which users already pressed the button, there's nothing preventing users from e.g. going in their browser's network inspector and repeating the request to increment the counter, despite the button being disabled (or using the inspector to reenable the button, etc. - the possibilities are endless).

For another example:
<ul>
  Your application has users play a simple fruit-themed platformer game.
  When they finish the level, a remote method is invoked to submit their score to the server.
  When all players finish, everyone's scores are displayed.
</ul>

In this case, there is nothing preventing a player from doctoring requests such that they end up being registered with an _illegitimate_ high score.
You would need to add dedicated logic to your application - probably both the server and client aspects - to make this sort of cheating a bit harder to achieve.

In general, whenever application server scripts receive method invocations and events from clients, they should be "skeptical" about the received message and the circumstances in which it was sent.
The application's server logic should be the source of truth and the entity performing the necessary checks on the state of the application. (In multiplayer games, this is what is known as the server-authoritative model.)
Motivated or simply curious users can easily invoke methods and emit events with the incorrect number of arguments, or with nonsensical data that does not match what your code expects.

> **Aside**: while it is true that the JAF could have protections to make this kind of "illegitimate message" harder to produce - e.g. by associating nonces and signatures with each one - in practice, this is an endless cat-and-mouse game: it will always be possible for a sufficiently motivated attacker to create custom messages that look legitimate from the point of view of the JAF - e.g. by extracting the parameters of said signature from within the JungleTV client SPA.
The fixed overhead of the additional checks would turn out be worthless, and likely irrelevant for many messages of the more trivial kind.
Only your application can perform the most relevant information integrity checks, because it is the one with knowledge of its domain and awareness of the performance requirements for the processing of each message.

### Global rate limiter

To preserve the integrity of the JungleTV service and the JAF runtime, incoming events and remote method invocations are subject to a shared rate limit on the server, of 60 events or method invocations per second, per origin IP address.
This limit is shared among all events/remote methods of all JAF applications running within a JungleTV environment.
If an origin IP address reaches this limit, they will receive an error when attempting to invoke remote methods or emit events to the server, and no event or invocation will be communicated to the destination JAF application.

> **Note**: right now, an exceeded rate limit resets immediately on the next second, but this is subject to change in future JAF updates.

To prevent running into this global rate limit, a good rule of thumb is to avoid sending more than 10 events per second from each instance (i.e. "browser tab") of an application page.
This is because multiple application pages may be loaded on the JungleTV client simultaneously, and it is important to be courteous to other applications.
