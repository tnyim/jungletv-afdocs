# Framework overview

The [introduction](../introduction/README.md#what-is-the-jungletv-application-framework) already explained _what_ the JungleTV Application Framework aims to be - a sort of official "game modding environment for JungleTV," with each mod being called an "application."
This section will briefly explain _how_ the JAF achieves that, and explain the most important concepts associated with it, so that you can understand the forthcoming sections.

## Concepts: server, client and environment

JungleTV is composed by two main pieces of software: its **server** and its **client**.
The server, which runs on some computer connected to the internet, is responsible for knowing e.g. what is playing at any moment, which media entries are in the queue, deciding whether users can send messages in chat, etc. and is also responsible for serving the client software when users access the website with their browsers.

This client software is essentially a web page with multiple megabytes of JavaScript, and which you'll often see referred as the **JungleTV SPA**.
In addition to providing the familiar JungleTV user interface, the client also communicates with the server, to obtain all of the information the server keeps track of, and to submit any requests (such as the aforementioned chat messages) for the server to process.

Generally when people mention JungleTV, they are thinking about the production version of the website at jungletv.live.
However, other JungleTV **environments** exist - different instances of the JungleTV server which have their own separate state, including a separate database, separate configuration, and their own client configured to connect to that different instance.
They may be available over the internet, or just in a local network.
In fact, because the JungleTV source code is available to the public, it is possible for you to run a JungleTV environment in your own machine (note that the project's license only allows you to run it in a private setting, not available to the wider public).

Henceforth, when we talk about "JungleTV" or its server and client, we won't be specifically mentioning _the_ official production version at jungletv.live, but also any other environment running software which is largely that which powers jungletv.live.

## Concepts: applications, editors, pages and files

**Applications** are computer programs that execute within the JungleTV server.
They can be written in JavaScript or [TypeScript](https://www.typescriptlang.org/), and they have access to special modules (libraries/packages) that let them interface and interact with different components and aspects of JungleTV.
Applications are developed by people such as you, who may or may not be part of the JungleTV team.

> **Aside**: In technical terms, there isn't a hard boundary between what constitutes the JungleTV Application Framework and what constitutes the general JungleTV functionality.
In practical terms, the JungleTV AF encompasses everything that was added or modified in the JungleTV software so that it could run these applications.

Applications are also able to add their own **pages** to the JungleTV user interface, which may be fully independent, like the About page or the Leaderboards page, or appear as part of smaller portions of the UI - much like the Chat and Queue tabs of the homepage sidebar, which are just a smaller part of what the users see on the homepage.
In many ways, pages are quite similar to their own isolated website, and can make use of mostly the same browser capabilities as one.
Pages, too, have access to special code that allows them to integrate with the rest of the JungleTV website, as well as communicate with the portions of application code running on the JungleTV server.

Applications are made up of **files**, among which may be the aforementioned JavaScript or TypeScript source code, as well as HTML markup, CSS sheets, images, fonts and other resources for any pages the application might want to add to the JungleTV website.
Files exist "inside" of the JungleTV server, like applications in general - they are stored in the database used by the server.

Files are uploaded to a JungleTV environment and managed through an **editor** - the JungleTV client includes its own simple application editor, but there are also external editors - as we'll see in the upcoming [development tools](./tools.md) section.
Files may be marked as available for download, served by the JungleTV server much like what happens with the resources needed by the JungleTV SPA.

## Capabilities and limitations

The JungleTV AF is not designed to let applications do _everything_ with the JungleTV service - on their own, they can't make JungleTV ditch Banano and have it distribute actual money, for example.
The capabilities of the framework are limited by what the aforementioned "special modules" are designed to let applications do, the general computational power limits available to applications and the JungleTV server as a whole, and _the subjective opinion of the JungleTV team about which kinds of applications and features they don't mind seeing on the service_.

Simultaneously, there isn't really a limit to the number of ideas which may be implemented while "painting inside the lines."
Here are some of the things that JAF applications can do:

- Add their own pages to the JungleTV website, including to specific predetermined portions of the UI - as explained previously;
  - Including adding new tabs to the homepage sidebar;
- Read chat messages and send messages in chat;
- Change the JP balance of users;
- Persist data so it remains saved even if the application or the JungleTV server restarts;
- Temporarily change the website logo and name, for special events.
- _List subject to expand as the JungleTV AF is developed._

And here are some of the things that JAF applications will most likely never be allowed to do:
- Replace the entire website with a completely different service;
- Add advertisements to JungleTV;
- Mine cryptocurrency or other bruteforce algorithms using the compute power of the JungleTV users;
- Adversely affect the execution of other JAF applications.

> **Note**: a more complete list of limitations, and the general intention behind the design of the JAF, can be found [in the Architecture manual section](../manual/architecture.md#limitations).