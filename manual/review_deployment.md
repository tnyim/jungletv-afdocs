# Application review and deployment

Once developers deem their JungleTV AF applications sufficiently ready for prime time, the projects should be submitted to a review process which, ideally, results in them becoming available on JungleTV proper, for the JungleTV community to enjoy.
This process is led by the JungleTV team, with the collaboration of the application developers.
Application review and deployment presently has no cost to application developers.

Both the application developers and the JungleTV team should be aware of a set of rights and duties, particularly around the handling of the Intellectual Property (IP) associated with the submitted JAF applications.
The following sections go into detail on what the review process encompasses, how the application IP is handled, what happens post-deployment, and the projected timeline for the processes.

## Review

The proper application review process begins when an application, or update to an existing application, is [submitted](#submitting-an-application) for review, regardless of whether informal comments may have been previously relayed to the application developers during e.g. public testing sessions in which the JungleTV team members participated.

The JungleTV team will then test the application in a private JungleTV environment, and inspect its files to find issues within the [scope of the review process](#scope-of-the-review).
This may take between a few days and a few weeks, depending on the size and scope of the application.
The review process will naturally be shorter for simpler applications.

Finally, the review process is concluded with the creation and delivery of a [results report](#results-report) to the application developer, containing a verdict on whether the application is ready for [deployment](./review_deployment.md) as well as a list of issues and recommendations.

### Submitting an application

Before submitting an application, make sure that all of the files contained within it are files you are comfortable with submitting and are relevant for the operation of the application.

In addition to the files needed for the application to function, you should create a plain-text file called `README` within your application, where you:
- Briefly explain in human-friendly English what the application is about, including a list of its notable features;
- Indicate the desired ID for the application (potentially user-visible in application page URLs);
- Include any special operation instructions that the JungleTV team should be aware of;
- Include stable contact information (including, at least, an email address) that the JungleTV team can use to reach you regarding your application;
- If your application uses licensed third-party assets, you should include all relevant licensing information and references, so that the JungleTV team can reference them in the event that a complaint related to intellectual property reaches the JungleTV team.

Once this is done, you should [export](./import_export.md#exporting-applications) the application from the environment where it is being developed.
For most applications, the resulting ZIP file is everything you need to submit.
However, if your application contains code that is not easily human-readable, for example, because it is the output of a minifier or preprocessor, you also need to attach the original unobfuscated source code, or point towards where it can be found (even if it is written in a language unsupported by the JAF).
This is strictly so that the reviewers can confirm that nothing suspect is happening within the terse final output.

The submission process is exactly the same for updates to previously submitted applications, but to avoid any doubts, you should also indicate that you are submitting an update rather than a new application.
As a courtesy to the reviewers, you should also include a summary of the changes in the update.

Application submission happens by sending an email with the exported application and the required information to the email address `jungletv -at- tny -dot- im`.
The JungleTV team will normally look at all submissions and respond with a [report](#results-report), but we reserve the right to ignore submissions that don't meet the criteria specified in this section, or from developers whose behavior has been considered abusive (for example, after repeatedly submitting what is effectively an unchanged application, despite an earlier rejection in the submission process).

To sum things up:
- Create `README` file **within the application**, with the desired application ID, a brief application description, special operation instructions if relevant, developer contact, and licensing information;
- [Export](./import_export.md#exporting-applications) the application;
- Send email, attaching the exported ZIP file.
  - If this is an update to a previously submitted application, you should briefly explain the changes since the last submission;
  - If there is minified code within your application, you will need to attach or reference the original source for those **outside of the application ZIP**.

### Scope of the review

Application review is not a general quality assurance process; as part of the review [results report](#results-report), the reviewer may let you know about general quality problems that they happened to run across, but the review is mainly focused on the following aspects:

- Ensuring that the application generally matches the description and behavior indicated by the developer.
  For example, ensuring that an application described to provide a polls feature is not actually a platformer multiplayer game;
- Ensuring that the application does not provide any "negatively surprising" features or functionality deemed as undesirable.
  For example, making sure that a game that lets users play games of chance to spend and earn JP, is not actually a subtle advert for a real money casino;
- Identifying major problems that may jeopardize the security or performance of the JungleTV service, or put the users at risk.
  For example, ensuring that the application does not consume unreasonable amounts of system resources, or that it does not steal the authentication tokens of the users.
- Verifying the application's use of JungleTV features like the [JP system](./jp.md) or the [chat](./chat.md), to ensure that these shared mediums are not abused;
- Confirming that, where appropriate, all user-facing interactive aspects of the application are accessible from mobile devices and devices with touch screens.
  The stringency on this point will depend greatly on the intended use cases for each aspect of the application; for example, we are typically more lenient when it comes to staff-only dashboards;
- Cursory spelling and grammar checks, i.e. essentially making sure no egregious typos or bad grammar are present on the most user-visible aspects of the application;
- Finally, understanding whether the application is a good fit for the JungleTV community, on subjective terms - i.e. whether the JungleTV staff sufficiently likes the application and is comfortable with its deployment on the service.
  - Despite the inherent subjectivity associated with this aspect of the evaluation, the JungleTV team vows to still provide a justification as to why an application is considered not to be a good fit.
  - Understandably, developing a complex project to then fail on this step would be frustrating for everyone involved, which is why it is a good idea to discuss its general concept with the JungleTV team prior to starting development, or during the early development phases.

Notably, the review process will normally not focus on:
- UI design aspects, unless they pose major usability issues;
- Code quality or code organization decisions;
- The quality and content of the creative assets (e.g. pictures, icons, sound, text) used, or whether the application developers have the right to use them;
  - As will be explained in a later section, the JungleTV team reserves the right to, at any point, unapprove any application for which they receive a complaint related to intellectual property usage rights;
  - Content will still be screened for the presence of NSFW/adult-only/material that the JungleTV team otherwise deems inappropriate;
- Subjective aspects unrelated to the application's interaction with the JungleTV components, e.g. whether a game is "fun" or whether a specific color scheme is "nice looking" (insofar as the usability and accessibility are not impacted).

Application developers should understand that the review process is not infallible, and problems _will_ slip by - potentially, even problems that may result in the application losing its approval post-deployment.
As those problems are identified, the JungleTV team may request that the developer make changes to the application, or they may make those changes themselves.
See the [Rights and duties](#rights-and-duties) section for more information.

### Results report

Once the JungleTV team is done processing your submission, you will receive an email with the results of the review.
In addition to communicating whether the application was approved or not, it will also include:
- If rejected on "bad fit" grounds, any fundamental concerns from the JungleTV team which prevent an application like the submitted one from ever being approved.
  This points towards a fundamental incompatibility of the application's concept with the team's vision for JungleTV;
- Identified issues with the application which prevent approval, and can only be fixed by the application developer;
- Blocking issues that the JungleTV team already fixed, which is more likely to happen for small changes in projects with sufficiently documented/organized code.
  You must incorporate these changes on your side before submitting further application updates;
- Minor non-blocking issues and suggestions - small details that should ideally be resolved in a future update, but which do not prevent application approval.

If approved, you will need to respond to this report, confirming that you authorize the [deployment](#deployment) of the application, considering any changes the JungleTV team may have made.
Even if you have received an approval, you may choose to not deploy at this stage, and fix any minor issues raised in the report.

If rejected, on other than "bad fit" grounds, you will need to make changes before submitting the application for review again, in order to fix the identified issues which prevented the approval.
For a given project, the subsequent review turn around times will typically be much shorter - this applies both to resubmissions after a rejection, as well as updates to previously approved applications.

### Rights and duties

With the common case of APIs offered through internet protocols like HTTP, there is a relatively clear technical separation between the first-party system offering the API, and the third-party systems that interact with it.
This clear separation extends to the ownership of the intellectual property (IP) of the third-party systems interacting with the API, with the first-party usually having no contact with the internal aspects of that IP, beyond any details disclosed through the very nature of the API interactions.

However, the JungleTV AF is [not a conventional web API offering](./architecture.md#the-missing-regular-api), with IP produced by third parties, including code and assets, being executed within, and served from, the JungleTV server.
This, together with the described application review process, means that the vast majority of the applications' IP is necessarily acessible to the JungleTV team.
Clarifying what rights are transferred to the JungleTV team when an application is submitted is, therefore, of utmost importance.

The JungleTV team does not claim ownership of the intellectual property of submitted applications.
The JungleTV team shall not misrepresent the authorship of the applications, never removing any credits or attribution present within.
The JungleTV team does not claim any exclusive rights to the applications; the application developers are free to continue using the applications' IP for purposes unrelated to JungleTV, including licensing the application to other third-parties.
The JungleTV team is not authorized to divulge details of the application's IP or the data produced by it, other than what is strictly necessary for the execution of the application as intended by its developers, unless compelled to by law enforcement.

By submitting an application developed for the JungleTV AF for review, the application developer grants the JungleTV team a limited license allowing for the analysis and modification of the application, and its execution in private JungleTV environments only, but not the redistribution to third-parties of the application, or any modifications made by the JungleTV team.
The modifications the JungleTV team is allowed to make to the application do not include the removal or modification of any credits or attribution.

There is an [additional set of rights and duties that applies post application deployment](#post-deployment-rights-and-duties).

#### Applications in public lab environments

Applications in public lab environments, such as the [staging lab environment](./environments_editors.md#the-staging-lab-environment), are owned by their developers.
They retain the copyright to the application IP in said environments, but grant the JungleTV team (or the operator of the lab environment) the right to execute the application, process and transmit the data produced by it, as far as necessary for the operation of the lab environment or the application itself.

The application developer must understand that applications in public lab environments are at a higher risk of being looked at, inspected, reverse-engineered or even tampered with, by other application developers; this is discouraged by the [staging environment etiquette](./environments_editors.md#staging-environment-etiquette) and the JungleTV team will attempt to remove from public lab environments those who violate these guidelines, but ultimately will not be held responsible for any damage caused by third-parties to the applications and intellectual property of other third-parties who choose to use public lab environments.

## Deployment

Once an application passes the review process, the JungleTV team is not yet authorized to deploy it to the production JungleTV site.
Once you, the application developer, authorize the deployment, the application becomes available on the JungleTV service at the sole discretion of the JungleTV team.

### Application launching

Some applications are meant to be long-running, essentially executing all of the time, indefinitely, or for a specific time period.
Others are meant to be executed for a brief moment, perhaps to perform some sort of data analysis.

In the production environment, application developers are unable to launch and stop their applications themselves; only the JungleTV team is able to do so.
If this is inadequate for your application, you should design it so that it can be executing all of the time, and the application developer (or an otherwise specifically designated set of users, or perhaps even the application logic itself based on predefined rules) is able to move it between different modes of operation, where one of those modes may be akin to a "standby mode" where the application has no visible effects.

The specifics of when an application should execute should be discussed with the JungleTV team upon authorizing the deployment of the application.

### Post-deployment rights and duties

By authorizing the deployment of an application that passed the review process, **in addition** to the [rights granted upon the submission of the application](#rights-and-duties) for review, the application developer grants the JungleTV team the right to execute that application in the public production JungleTV environment, presently hosted at [jungletv.live](https://jungletv.live).
The JungleTV team is also granted the right to process and distribute any data produced by the application as deemed necessary for the operation of the JungleTV service and/or the application, including for purposes of data backup, transfer between servers, incident investigation, auditing purposes or interaction between the client and server components of JungleTV.
Finally, the JungleTV team is also authorized to use the application for promotion of the JungleTV service, including listing it among the features or attractions of the service in marketing materials, attributing the application's authorship as the JungleTV team sees fit.

The application developer is granted the right to request the removal of the application from the production environment, effectively cancelling its deployment authorization, by sending an email to the address `jungletv -at- tny -dot- im` (the same one used for application submissions), from one of the contacts the JungleTV team has on file for the application developer.
This removes the right for the JungleTV team to execute the application or use it to promote the service, but the JungleTV team holds on to the right to process any data produced by the application prior to deployment, for incident investigation and auditing purposes.

The JungleTV team reserves the right to prevent or stop the execution of any application, and remove any application at any moment from the production environment, particularly if the application is found to present unexpected behavior, or contain security vulnerabilities.
