# Architecture overview

## JungleTV Architecture
Before delving into the architecture of JungleTV AF, let's briefly go over the architecture of JungleTV itself, which is relatively simple.
Like most web services, JungleTV fits into the client-server model, with communication happening over [HTTP/2](https://developer.mozilla.org/en-US/docs/Glossary/HTTP_2).
The diagram below shows how the different parts interact at a high level.

![JungleTV Architecture Diagram](../assets/manual/jungletv_architecture.drawio.svg)

The server is written in [Go](https://go.dev/) and runs as a single process based on a single executable.
The server typically sits in front of a [reverse proxy](https://en.wikipedia.org/wiki/Reverse_proxy), specifically Cloudflare.
It handles requests and simultaneous long-running connections from hundreds of clients, i.e. website visitors, and keeps state such as the currently playing media, the entries in queue, the messages in chat, etc., mutating this state according to the business logic of the service.

> **Aside**: There are some auxiliary server functions that can run as separate processes in the same or different machines, using separate binaries. The server is also a client for _upstream_ services, like the Banano node RPC server, and other 3rd-party APIs like Tenor.

As in most systems that make use of the client-server model, the server is authoritative, that is, the decisions about "what happens and what doesn't" are made strictly by the server.
This is in contrast with, for example, some peer-to-peer applications where a consensus must be reached between the different participants for "something to happen" - as is the case with blockchains; or some peer-to-peer games where notably a consensus does _not_ need to be reached between game clients, leading to rampant cheating and other shenanigans that should be "impossible" according to the intended business logic.

The JungleTV client software is a JavaScript-heavy [Single-Page Application (SPA)](https://developer.mozilla.org/en-US/docs/Glossary/SPA) that runs in the browser of the visitors.
It is developed using [Svelte](https://svelte.dev/) and [TypeScript](https://www.typescriptlang.org/).
Communication between the clients and the server happens over a [non-standard gRPC Web implementation](https://github.com/improbable-eng/grpc-web/), that is, a way to use [gRPC](https://grpc.io/) in the context of a web page.
This allows for real-time communication between the server and the clients, with pretty much the same capabilities as those offered by [WebSockets](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API).
In fact, gRPC Web can be configured to use WebSockets as transport instead of HTTP/2.

> **Aside**: gRPC structures and abstracts the communication, making it easier to write servers and clients that conform to the same API.
> The relevant groups of standards have recently expanded to also allow gRPC to be used in the browser, in the context of a web page.
> The non-standard implementation used by JungleTV predates these standards and is slightly different in ways that make it incompatible.
> The fitness of the used implementation has been thoroughly verified through years of JungleTV operation, so moving to the standard implementation is low on the list of priorities, but it may happen eventually.

Despite being a Remote Procedure Call (RPC) framework, gRPC can be used in many different architectural styles beyond client-server, including event-driven / implicit invocation and service-oriented architectures (gRPC is commonly used for communication between microservices in microservice-based service-oriented architectures).
To define the JungleTV architecture in fancy terms, it is a client-server distributed application, with both the server and client being component-based, event-driven monolithic applications.

If all of this sounds overwhelming, worry not: the JungleTV AF is designed precisely to allow developers to add functionality to JungleTV without having to deeply understand the technology stack and internal implementation of the service.

## The Application Framework

The core of the JungleTV Application Framework is, fundamentally, a JavaScript (ECMAScript) runtime within the JungleTV server.
Code running within this runtime can invoke _native_ code, that is, code that is part of the JungleTV server, written in Go and "set in stone" (i.e. not alterable by AF applications).
Native code is exposed to JavaScript through an interface, and lets applications manipulate different aspects of the server and service, in a loosely controlled fashion - just like the JavaScript browser APIs let web pages invoke some of the native code of the web browsers, and e.g. open an alert through `window.alert()`, or just like Node.js lets JavaScript code open files through `fs.readFile()`.

There are other components that make up the JAF, including an entire file storage and management system (on the server), code for dealing with application pages (on the client) and a script that client-side application code can make use of, in order to interact with the JungleTV client (the appbridge).
The diagram below shows an overview of where each JAF component lives and how the different aspects interact.

![JungleTV AF Overview Diagram](../assets/manual/jungletvaf_overview.drawio.svg)

The JavaScript runtime used in the JungleTV server is [Goja](https://github.com/dop251/goja), augmented with JungleTV-specific native modules as well as an [event loop](https://github.com/dop251/goja_nodejs/tree/master/eventloop).
In addition to the "classic" functions of the `setInterval`/`setTimeout` family, the event loop enriches Goja with the ability to support async/await and we encourage application developers to use this to their advantage.
Each JungleTV application gets a separate instance of Goja, with a separate event loop.
Some aspects of the native modules are directly or indirectly shared among applications - after all, there are multiple applications but there is a single chat, a single queue, and so on, per JungleTV environment.

While Goja is fast compared to other JavaScript engines written in pure Go, it is orders of magnitude slower than e.g. JIT compiling engines such as V8, used in Node.
You will find that Goja is still plenty fast for all intended use cases, but these limitations mean that writing your own raytracer within application server scripts is unlikely to render a good result, performance-wise.

### No True Sandboxing

While the Goja runtime provides a semblance of a sandbox, it is definitely not as hardened or battle-tested as the sandboxes found in web browsers.
There is no process isolation between JAF applications; the CPU and memory usage of JAF applications is not metered or limited in any way (largely because Goja does not offer functionality to such effect).
This means it is quite easy for a JAF server script to adversely affect the performance and availability of the JungleTV service as a whole.
This is why application code must be readable to JungleTV staff and it is why applications go through a [review process](./review_deployment.md) before being deployed to the actual JungleTV website.

Similarly, while the code of the native modules available to JAF server scripts makes an effort to validate their usages, e.g. by warning about missing or invalid parameters in function calls, it is not built to withstand malicious intent.

In conclusion, the server code of JAF applications is considered _trusted_ in the JungleTV security model - or in other words, in theory it is able to do _anything_ just like the native Go code and _if_ malicious code were to execute within the JAF runtime, the system would do very little to prevent any damage.
All of the so-called "protections" in place are present mainly as a convenience for application developers - for example, the detection of a stuck event loop - and not as a security measure.
In practice, you will find that many things are impossible without "breaking the rules", as explained in the next section.

### Limitations

The main purpose of the JungleTV AF is to greatly decrease the barrier to entry when it comes to _adding functionality_ to the service.
While application sandboxing isn't an actual part of the system, _by design_ the JungleTV AF implies some limitations on what applications are able to do.
The lack of a "real sandbox" means that _technically_ JAF applications can likely bypass such limitations, but such "creative solutions" may still be turned back through subjective human review - much like technicalities in how laws are written often don't hold up in court.
Therefore, just like how with actual laws it is important to understand the intent of the lawmaker, it is important to understand the intent of the JAF limitations.

JAF applications are meant to _extend_, not _replace_ or _wholly redesign_, the JungleTV service.
Therefore, the JAF does not provide the means for applications to completely change the look of the JungleTV client SPA, or to fundamentally change how most built-in features work.
When in doubt, you should consult JungleTV staff on whether your ideas and use cases constitute an acceptable use of the JAF.
Here are some non-exhaustive examples of things JAF applications are **not** meant to do:

- Manipulate files in the machine the JungleTV server runs on.
- Replace the entire website with a completely different service, or otherwise redirect all users to a different website.
- Make HTTP requests or other internet connections to external services. \*
- Make the website have multiple queues or chat rooms with the same appearance as the built-in ones.
- Run compute-heavy algorithms on the server, for instance, implement a chat bot based on a Large Language Model running within the AF runtime.
- Add support for new types of media that gets enqueued through the usual built-in flow. \*
- Collect "service fees" that revert towards the application developer.
- Divert all participation rewards to a subset of users affiliated with the application developer.
- Add advertisements to JungleTV.
- Mine cryptocurrency or other bruteforce algorithms using the compute power of the JungleTV users.
- Adversely affect the execution of other JAF applications.

The main goal of these limitations is to protect both the data of the JungleTV users, the integrity/availability of the service itself, and the general "vision" the founders have for the service.
They tie in heavily with the [review process](./review_deployment.md).
Many limitations may disappear - especially the ones marked with \*, possibly on a case-by-case basis, as the Application Framework evolves and as application developers come up with creative ideas and "proofs of concept."
It is also important to note that, even after the JAF becomes extremely mature, some feature ideas may still be best implemented as part of the JungleTV core, rather than as JAF applications.

Finally, not all limitations of the JAF architecture are intentional, but more of a compromise.
For one, developers become limited in the choice of programming language, having to effectively use JavaScript or TypeScript for the server-side logic.
If this ends up being too much of a limiting factor, in the long term we may explore offering other runtime options, like WASM, that would allow a wider choice of technologies.

## The missing "regular API"

**Why doesn't JungleTV simply offer a plain old REST API with which external systems can interact, rather than making the execution of all 3rd-party extensions happen entirely inside the domain of JungleTV?**

Providing a public REST API would likely have been much simpler than building the Application Framework, at least in the short term.
There are multiple reasons why we decided to go with an approach that is decidedly more complex and less common:

- **Lower latency on user interactions**: in JungleTV, users interact with the service and among themselves in a highly synchronous, real-time fashion.
  This is also true for many other services that offer a more conventional API, such as Discord or Twitch.
  You may have noticed how, on such services, sometimes the interaction with 3rd-party extensions (such as Discord bots) can feel a bit sluggish.
  This is because communication of the users' actions must go from the users' machines to the service, and from the service to the third party, and any result must also go through the same two hops on the way back.
  Because on the JAF the decision-making happens entirely inside the JungleTV server, the latency between JungleTV and the 3rd-party code is basically zero.
  This should allow for certain interactive experiences to be developed by third parties, that would not be possible with higher latencies.
  Consider also that the JungleTV backend is not globally distributed like that of e.g. Discord, so the more traditional approach could incur even higher latency penalties, depending on the geographical location of the 3rd-party systems.
- **Reliability of the experience**: extensions to JungleTV that depend on a 3rd-party system will always tend to be less reliable than those built into the core of JungleTV, simply because all the possible points of failure within JungleTV get compounded with all those within the 3rd-party system.
  When a part of the JungleTV service runs on a separate computer system that must be reached over a network, we must consider many additional possible points of failure.
  Running the extensions to JungleTV within the JungleTV server itself eliminates some points of failure, namely all of those that depend on the reliability of the connection to an external machine or the availability of said machine.
- **Lowering some barriers to entry**: connected with the previous point, the JAF architecture means that application developers do not need to operate a reliable, long-running machine to serve an extension to the JungleTV service.
  JungleTV effectively hosts the JAF applications at no additional cost to their developers.
  Applications do not become unavailable because something broke on the developers' servers and nobody on their team is awake to fix them.
  Additionally, getting started is simpler, as users do not need to download or configure any software on their machines to develop applications: with the online editors, developers can effectively begin working on JAF applications even from more limited devices like tablets, Chromebooks and shared computers.
- **Protection of user data and privacy**: by keeping execution of 3rd-party code within the domain of JungleTV, and by performing a light vetting of that code prior to production deployment, we can be reasonably sure that the data going through the service is not unduly collected or used for any unexpected purposes.
- **Easier long-term management of backwards compatibility**: especially as the JungleTV AF is in its earlier phases, it is likely that some aspects will change over time.
  By hosting the applications on JungleTV itself, the JungleTV team can more easily understand what specific features and APIs are being used by JAF applications.
  If allowed by application developers, the JungleTV team can even perform quick fixes to applications so they remain compatible as the JAF evolves, even if the application developers have little availability or interest in keeping up with those changes.

It goes without saying that the lack of a "plain old public API" constrains developers in some ways: for example, they can't use their technology stack of choice, being limited to what is supported by the JAF.
Additionally, the fact that application code goes through a [minimal review](./review_deployment.md) before production deployment, means that developers may feel pressured to make their code understandable to others, which might not be to everyone's liking.
Still, considering the use cases we envision for the JungleTV Application Framework, we believe that this more complex model is a better fit than a regular API for external consumption.