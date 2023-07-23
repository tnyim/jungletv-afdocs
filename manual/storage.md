# Data storage

The architecture of the JungleTV Application Framework encourages a separation between the static assets of an application, including its code, and the data produced and manipulated by it.
Server-side code can't directly access [application files](./applications_and_files.md#application-files), with the exception of JSON files and code that can be imported.
Simultaneously, many applications need to persist data across different executions; even applications meant to be long-running may need to do so, because interruptions to an [application's lifecycle](./applications_and_files.md#application-lifecycle) may be unexpected.
For example, an application that allows users to vote on polls will likely want to store the tallies of ongoing and closed polls across executions.
If application files can't be manipulated by the applications themselves, it follows that different mechanisms must be used to persist data from within application instances.

## Key-value storage

Presently, JAF applications have access to a single, rather basic, server-side data storage mechanism, provided through the [`jungletv:keyvalue` module](../reference/server/jungletv_keyvalue.md).
This is a simple key-value storage that is private to each application, meaning that applications can't access the data stored by other applications.
The keys and values are stored as strings.
It operates very similarly to the [Web Storage API](https://developer.mozilla.org/en-US/docs/Web/API/Storage), that is the one used for local storage in browser contexts.

> **Aside**: internally, each key-value pair corresponds to a row in a JungleTV database table that is indexed by both application ID and key name.
> Operations on the key-value storage directly result in database queries, and therefore have more of a performance overhead than in-memory data accesses.
> Because these operations are still expected to be very fast, on the order of a couple milliseconds at most, for simplicity the `jungletv:keyvalue` API is synchronous.

Saving a value to storage is extremely simple.
Let's imagine that we want to persist some sort of counter value:

```js
// Server code
const keyvalue = require("jungletv:keyvalue");

let counter = 123;

keyvalue.setItem("my counter", counter);
```

Retrieving this value from storage would also be very simple, although one must take into account that all values are converted to strings when saved, and thus may need to be converted back to the correct type when read:

```js
counter = parseInt(keyvalue.getItem("my counter"));
```

It is also possible to enumerate the key-value pairs saved by the application, and completely remove one from the storage.
See the [`jungletv:keyvalue` module reference](../reference/server/jungletv_keyvalue.md) for details.

## Browser local storage

Client-side application code is allowed to use the [`localStorage`](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage) and [`sessionStorage`](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage) to store data that is best kept associated with the browser or a specific JungleTV page session, such as the UI preferences for some application page.
(A page session is not to be confused with a browser session: see the `sessionStorage` documentation for details.)
Data that should be associated with a JungleTV user, rather than a browser, should be sent to the server and stored there.

To avoid collisions with other applications and JungleTV itself, it is a good practice to prefix storage keys with the application ID or some other string that ensures the values won't be accidentally read or overwritten by pages belonging to other applications.

> **Note**: due to how JungleTV application pages are served and the iframe sandboxing options used, the local storage for application pages is shared among applications and with the JungleTV website itself.
> Therefore, application pages have access to the users' authentication tokens for the website.
> The application [review process](./review_deployment.md) pays special attention to the code of application pages to ensure that nothing nefarious happens with this unfettered access to the authentication tokens.
> This is in line with the [No True Sandboxing](./architecture.md#no-true-sandboxing) philosophy that permeates the JungleTV AF.

Of course, application pages also have access to other client-side storage mechanisms, including the good old [cookies](https://developer.mozilla.org/en-US/docs/web/api/document/cookie) and [IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API/Using_IndexedDB).
The same concerns around avoiding collisions with other applications and undue access to sensitive information apply all the same, regardless of the specific technology or API used by application pages for browser local storage.