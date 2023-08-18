# Application testing

Testing is one of the core activities of software development.
Software systems can be tested and evaluated in many different dimensions - including functionality, stability, scalability, accessibility and security - to understand whether they conform to a specification or set of requirements, to ascertain their fitness for a particular purpose, to estimate the cost associated with their usage, to identify vulnerabilities, check their compatibility with other systems, or a multitude of other motivations.
Software testing can use an equally large number of techniques and approaches; some can be automated, typically through the use of additional software/code.

When it comes to JungleTV AF applications, one is typically interested in ensuring that:

- The features of the application work as expected/designed by its developers (functional testing).
- The application is secure - that is, it does not jeopardize the integrity, confidentiality, or availability of its own functionality/data, nor those of JungleTV as a whole (security testing).
- The application is _usable by its intended users_, i.e. those for whom the application is designed can operate it without trouble (usability testing).
- The application's performance is sufficient to withstand the expected number of simultaneous users, and its resource consumption in line with what's expected for the functionality it provides (performance testing).

The JAF does not presently include any functionality specifically designed to aid in application testing, but this does not mean that testing - of both the automated and manual kinds - is impossible.
The end goal of application developers should be to _avoid surprises in production_: by the time the application is developed on the production JungleTV website and possibly being used by hundreds of users, it should work as envisioned by its developers, and there should be no occurence that comes across as unexpected to either the application developers or JungleTV staff.
In the following sections, we will give some ideas on how to thoroughly test JAF applications in the most relevant dimensions.

## Functional testing

In colloquial terms, this is the process of "finding bugs" in the application.
For example: an application that lets users vote on polls must ensure that when a user votes for option B on a given poll, that vote is counted for that option on that poll, and not a different option or a different poll; a platformer game always registers the death of the player character when they fall into a pit; a link in an application page points to the right destination and not a missing or mismatched page; and so on.

Manually testing the application by launching it in a lab environment and going through its different features, like a normal user would, can catch the most obvious problems.
To uncover more issues, make sure to also go outside of the expected user flows and specifically test edge cases by, for example:
- Providing unexpected or boundary values in form fields;
- Using the application's user interface from devices with atypical screen dimensions, browser engines or input methods;
- Operating the application in uncommon JungleTV conditions (e.g. enqueuing disabled, chat disabled, or empty/very full media queue...);
- Visiting the application pages or otherwise using the application as an unauthenticated user;
- If the application integrates with the [chat](./chat.md), making sure that it can correctly handle e.g. messages with no content but attached media, or messages sent by other applications, as well as multi-line messages, messages with emojis and other "special characters";
- If the application makes use of the [JP system](./jp.md), making sure that it can deal with e.g. users with insufficient points balance to perform some action.

For simpler applications, this type of testing can be sufficient: the less complex the functionality of an application, the more likely it is that manual testing is able to identify all possible problems.

Especially for more complex projects, application developers are encouraged to have some form of automated testing.
This is made easier by keeping code organized, separating concerns across different code units, and introducing layers of abstraction where it makes sense.
One can then have a set of code units (e.g. functions or classes) dedicated to performing unit testing and integration testing of other units, which can be triggered by e.g. executing a specific command in the application console (or the web browser development tools, in the case of client-side scripts).

As an example, an application that lets users vote on polls may make use of an abstraction over the data storage mechanism that lets said mechanism be replaced during tests (e.g. through the use of a functional mock).
It can also isolate the core logic, that decides whether users are allowed to vote on a poll, is responsible for deduplicating votes, compute poll results, etc. such that it can be used both by RPC and event handlers as well as the automated test code.

Developers are free to bring in JavaScript testing frameworks as long as they do not depend on e.g. NodeJS-specific features or language features unsupported by [Goja](https://github.com/dop251/goja), the JavaScript engine used by the JungleTV server to run JAF applications.

## Security testing

[Security testing](https://en.wikipedia.org/wiki/Security_testing) is undoubtedly one of the more complex kinds of software testing, largely because it often consists on exploring the less visible aspects of a system, and thinking not as a typical user, but as an attacker trying to tamper with the system.
This often involves interacting with it in unconventional ways, for example, by using specialized tools to send specially-crafted requests to the server logic, rather than the application pages provided by a JAF application.

It is not feasible to expect JAF application developers to be knowledgeable and available enough to thoroughly penetration test their applications, nor to have the resources to contract a team to do so.
Simultaneously, the fact that JAF applications are contained within the larger JungleTV environment means that many metrics and methods typically used in "checklist-based" security evaluations may yield results that are irrelevant to the application developer.
For example:
- It is not up to the JAF applications to decide the HTTPS configuration used in the communication between their server and client parts;
- JAF applications have little to no control over the HTTP headers used in this communication;
- JungleTV itself needs to include scripts from third parties (e.g. YouTube) within its pages to provide its main functionality, which may have security considerations of their own, etc.

This means that most off-the-shelf automated approaches are also unlikely to be useful in the context of JAF applications and will, at best, deliver a report that commingles application, JAF and JungleTV concerns.

As part of the [review process](./review_deployment.md), the JungleTV team is on the lookout for glaring security issues with the applications, particularly those that could cause issues to the JungleTV service as a whole.
However, this isn't a thorough security evaluation by any means.
Should continued tests and production use of the application reveal the presence of security vulnerabilities directly caused by faulty application logic, the JungleTV team reserves the right to disable the application until the relevant problems are addressed.

There is no silver bullet we can provide, that drastically simplifies or eliminates the need for security testing.
At the same time, it is out of the scope of this manual to be a complete guide to web application security.
We recommend that developers with little experience with web development read a primer on the subject, such as [this one](https://developer.mozilla.org/en-US/docs/Learn/Server-side/First_steps/Website_security), and go from there.

## Usability testing

[Usability testing](https://en.wikipedia.org/wiki/Usability_testing) relates to understanding the capacity of a system to let its intended users perform tasks effectively and efficiently.
In the context of a JAF application, this consists on understanding whether its users are able to use it to achieve the objectives the application is meant to facilitate.
_Users_ are those for whom an application was developed, who might be a subset of the JungleTV community.
For example, an application meant to facilitate community moderation only needs to be considered usable by moderators.
Even an application with which the entire community interacts may have functionality only intended to be operated by a subset of users.

Usability is largely, but not just about the _user interface_ or _user experience_ aspects of a piece of software.
It also involves the feature set of said software; for example, an application that requires passing data across different devices but offers no option to download or otherwise transfer such data, like QR codes, will be considered less usable than one that offers such features.
Some also see usability as closely tied to accessibility.

This is one of the more subjective dimensions of software testing, because what is considered practical, safe or even "obvious" by one user, might not be considered so straightforward by a different user.
This may be due to matters of personal taste, education/prior experience, the context of the interactions with the system, etc.
There exist different models and techniques to evaluate the usability of a software system, some which involve analysing the system on a purely theoretical basis, and others which involve testing with users, in environments that may be more or less controlled, with more or less guidance provided to test subjects.

It is unreasonable to expect JAF application developers go to through extensive usability testing on their own.
What is expected, is for developers to be available to receive feedback regarding the usability of their applications, both before and after they are deployed in production.

In the context of the JAF, we recommend gathering a small set of users willing to participate in the testing of your applications, in an environment such as the [staging lab environment](./environments_editors.md#the-staging-lab-environment), giving them directions on the aspects you are looking to test or get feedback on, and repeating this process as the developer makes changes to the application.
Developers can prepare short surveys to present these users after each iteration, in order to measure the progress in a more streamlined, or even "objective," way.
This will let you perform both usability testing and functional testing, in the sense that the more the system gets used, the more likely it is for any remaining functional issues to be identified.
After some iterations of changes and feedback from a representative set of intended users, your application is much more likely to be considered highly usable by most who will interact with it.

## Performance testing

With the JungleTV AF presently collecting no metrics on application resource consumption, it can be tough to understand whether a certain solution will be able to scale to hundreds of simultaneous users.
This is an aspect where the JAF internals themselves are largely unproven, as aside from stress testing with simulated users, plus a few limited real-world tests, the framework's suitability for large-scale complex applications is still unknown.

In terms of raw compute performance, developers should feel free to perform quick benchmarks of their code in the public [staging lab environment](./environments_editors.md#the-staging-lab-environment).
Like with functional testing, this is made easier by the development of good abstractions and separation of concerns within the applications' code.
Developers can then test the core aspects of their server or client code by calling test benchmark functions that exercise such code, and measuring their average execution time.

With the use cases we envision for the JAF, we believe that most applications are unlikely to run into performance-related issues.
For more complex or unique ideas, should you have questions about whether a certain approach is feasible, or whether it would consume more resources than deemed appropriate, feel free to discuss it with JungleTV staff, who will also help you conduct performance testing if necessary - including deploying your experimental application in controlled environments, given that shared environments such as the staging one can be quite "noisy."

Like with security, performance is another aspect where the JungleTV team will make a quick analysis as part of the [review process](./review_deployment.md).
While we will let you know about any concerns, our failure to identify any possible performance problems naturally does not guarantee their absence.
Should continued tests and production use of the application reveal performance issues caused by poor application design or an inadequacy of the framework for the attempted use case, the JungleTV team reserves the right to disable the application.
