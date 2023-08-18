# Framework versioning and compatibility guarantees

The JungleTV Application Framework is expected to evolve over time and, at some point, changes may be introduced that cause previously developed applications to no longer function entirely as intended by their developers.
We will refer to those changes as breaking changes, and examples of such possible changes include:

- Renaming or removal of appbridge and server-side module functions;
- Changes to the meaning and number of arguments used by appbridge and module functions;
- Changes to the mandatory files an application must contain in order to be considered valid;
- A reduction to the number of supported languages and language features.

We will naturally attempt to minimize the number of breaking changes, but doing so may not always be feasible, particularly if said changes are required to mitigate security vulnerabilities.
Additionally, during these earlier stages of the Application Framework, there is a higher probability that breaking changes will be made, for the sake of consistency and stability in future versions.

There are also countless examples of non-breaking changes, i.e. those that would not cause any application to become incompatible, such as:
- The addition of new functions to server-side modules and to the appbridge script, as well as new optional arguments to existing functions, as long as the default behavior does not change;
- JavaScript runtime performance improvements;
- Support for additional programming languages...

## Obtaining version information

Although one JAF application version is not expected to be compatible with more than a single framework version, applications are nevertheless able to obtain information about the framework version they are executing in.

To obtain the main framework version which, beyond version 1, is only expected to change when breaking changes occur to either the server-side aspects, to the appbridge or client-server communication framework, applications can consult the server-side [`process.version` property](../reference/server/node_process.md#version).

To obtain the version of the appbridge script, client-side scripts can read the [`BRIDGE_VERSION` appbridge property](../reference/appbridge/api.md#bridge_version).
This version is expected to increase not only when breaking changes are made to the application-facing appbridge API, but also when changes are made that change the internal communication protocol between the appbridge and the host JungleTV SPA.
In general, application developers should ignore this version number and focus on the main framework version which, as will be explained in the next section, is the indicator of breaking changes that developers should be aware of.

## Compatibility guarantees

The JAF runtime version, as read through [`process.version`](../reference/server/node_process.md#version), will be increased every time there are breaking changes.
However, **this does not apply presently**, as until the JAF leaves version 1, breaking changes _may_ occur without any change to application-readable version numbers.
Version 1 should be understood as a beta/pre-release version of JAF, similar to the 0.x moment in semantic versioning.

Past version 1, the JungleTV team vows to not make any breaking change without a corresponding increment of the framework version number.
In general, we will try to avoid any breaking changes, and should those be necessary, we will attempt to modify previously [approved](./review_deployment.md#review) applications to fix any compatibility issues - hence why the JungleTV team reserves the right to modify applications submitted for review, as detailed in [a previous section](./review_deployment.md#rights-and-duties).
It is likely that the JungleTV team won't have the knowledge or availability necessary to fix more complex applications in the event of serious breaking changes being introduced in the Application Framework, at which point the JungleTV team will remove from the affected applications from the production environment and contact the application developer to kindly request the necessary adaptations.

## Changelog

Notable changes to the JAF including, but not limited to, any breaking changes, will be listed in this manual section, or in a resource that will be listed on this manual section, once the JAF moves past version 1.