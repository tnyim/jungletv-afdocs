# Application pages

Application pages are one of the primary methods for JungleTV AF applications to present a user interface.
In their most basic form, application pages are entirely new web pages within the JungleTV client SPA (i.e. "the website"), much like the leaderboards or rewards pages, but presenting content that is entirely application-defined.
Application pages can also manifest as extensions or overrides to specific components of the JungleTV SPA, where they will typically be displayed as just part of a larger section of the website's UI.

Application pages are built using standard web technologies and are able to use most capabilities available to web pages, subject to mild sandboxing.
Because they are just part of a larger website, they can't use features that depend on the configuration of a custom [web app manifest](https://developer.mozilla.org/en-US/docs/Web/Manifest).

## Publishing pages

Application pages are published during application execution, through the use of the [`publishFile()` function of the `jungletv:pages` module](../reference/server/jungletv_pages.md#publishfile).
Each application page must have a unique ID, that is up to the application to decide.
The ID must be URL-friendly and therefore certain limitations, documented in the function's reference, are enforced.
Published application pages can be accessed via the URL `https://jungletv.live/apps/applicationID/pageID`, where `applicationID` is the ID of the application that published the page, and `pageID` is the page ID specified.
The contents of the pages are somewhat isolated from the rest of the JungleTV SPA, as they are rendered inside an [`<iframe>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe).

To publish an application page, the application must contain at least one public HTML file, or another type of resource that can be used as the source for an iframe.
The use of an HTML file is strongly recommended, specifically, one that follows the minimum viable example shown below:

```html
<!DOCTYPE html>
<html lang="en-US">
<head>
    <script type="text/javascript" src="/build/appbridge.js"></script>
    <!-- ... -->
</head>
<body>
    <!-- ... -->
</body>
```

The script referenced in this minimal example corresponds to the [appbridge](#client-side-framework-appbridge); among other things, it is used for communication between the domain of the application page (inside of the iframe), and the domain of the JungleTV client SPA (outside of the iframe).
It is important to include this script because the JungleTV SPA is expecting to be able to communicate with the application page, and its failed repeated attempts to do may degrade the website's performance.
More details on the appbridge will follow on the next section.

> **Note**: If you so wish, a single application file can be published multiple times as pages with different IDs.
> The client scripts of the page may then assume different behaviors depending on the ID of the page.
> This is akin to how regular Single-Page Applications, like the JungleTV one, work.
> Typically the same HTML and JavaScript code is loaded regardless of the address bar URL, and then the scripts display different content based on the URL.

The default [Content Security Policy (CSP)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP) applied to application pages forbids inline scripts, i.e. JavaScript code within `<script>` tags on the HTML markup.
This encourages the generally good practice of keeping scripts and HTML markup separate.
If really necessary, a custom CSP may be specified when calling [`publishFile()`](../reference/server/jungletv_pages.md#publishfile).

> **Note**: the automatic compilation of TypeScript resources described in the [language support section](./languages.md#typescript-in-browser-contexts) will not apply to inline scripts.

Application pages can communicate both with the application server code, as well as the JungleTV client SPA, as will be explained in the upcoming sections [Client-server communication](./rpc.md) and [Client-side framework (appbridge)](#client-side-framework-appbridge), respectively.
Application pages have access to the browser local storage, as will be explained in the [data storage section](./storage.md#browser-local-storage).

Application pages that target the general audience of JungleTV, rather than just staff members (or some other privileged subset of users), should be mobile- and touch-friendly.
Generally, if your application pages offer a poor user experience to the point of being very hard to use for their target audience, that may reflect poorly on your application's [review process](./review_deployment.md).

When an application no longer wishes to make a page available, it can be unpublished through the [`unpublish()` function](../reference/server/jungletv_pages.md#unpublish).
For connected users visiting the page in its more standalone form, this will cause the page to instantly turn into a 404 screen.
If the page is being presented in a more integrated way within the JungleTV SPA UI, this typically makes the UI revert to a state similar to the one preceding the UI reconfiguration - see the forthcoming [Showing pages as part of the main UI](#showing-pages-as-part-of-the-main-ui) section for details.

## Referencing other application resources

Application pages can _transparently_ reference other **public** files of the application, such as scripts for execution in the browser, CSS styles, images, and so on.
This is thanks to the previously described JAF ability to [serve application files over HTTPS](./applications_and_files.md#serving-files-over-https).
The "transparent" aspect is due to how simple and seamless these references are.
Just reference resources using relative paths, because the files are on the same path as the one the HTML markup is served from.
It's exactly the same as on any static website that keeps all of its files in a single folder.

Imagine, for example, that your application has both an `example.js` script you want to include, and an `image.png` to show on the page:

```html
<!DOCTYPE html>
<html lang="en-US">
<head>
    <script type="text/javascript" src="/build/appbridge.js"></script>
    <script type="text/javascript" src="example.js"></script>
</head>
<body>
    <img src="image.png" alt="Some image" />
</body>
```

> **Aside**: if you are curious about how this is implemented, then you can figure this out using the inspector within your browser: a look into how the application pages are being loaded in the iframe's `src` should let you identify a connection between this ability to reference files using relative paths to the same directory, and a previously mentioned limitation on application file names ✳😉.

If you are having trouble loading application files within an application page, ensure:
- That the files are marked as public;
- That the MIME types of the files are set correctly;
- That you are referencing files from the right application.
  The JungleTV AF is only prepared to reliably let application pages serve files belonging to the application that published the page.
- That you are not running into limitations enforced by the default resource and permission policy for application pages.
  The policies can be partially overriden through the specification of custom headers when calling [`publishFile()`](../reference/server/jungletv_pages.md#publishfile).

## Client-side framework (appbridge)

The script included in the minimal application page example is essential to unlock most of the abilities available to application pages.
Through this script, application pages are able to communicate with the "host" JungleTV client SPA, as well as with the server-side logic of their application.
The JungleTV AF expects all application pages to load this script in the exact way shown in the minimal example: in the page's `<head>`, with no deferred loading or any other changes.

Client-side application code interacts with the appbridge script through the global object `appbridge`, accessible through e.g. `window.appbridge`.
This object's API is documented in the [appbridge reference section](../reference/appbridge/README.md).
Most functions are [asynchronous](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function), i.e. they return a [`Promise`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises) and can therefore be awaited (with the `await` keyword).

> **Tip**: once you've published and opened an application page, you may use your browser inspector tools to explore the appbridge API through the JavaScript console.
> Just make sure to set the context of the console to the page inside the iframe.

Scripts can listen to events pertaining to the "host" page, by registering event handlers on the [`page` object](#) **[[TODO] Link to correct reference once written]**.

The appbridge capabilities and functionality include:

- Obtaining the relevant application and page IDs;
- Understanding whether the user is authenticated on the website, and obtaining their reward address and privilege level;
- Navigating to different pages within the JungleTV SPA - including both pages of other applications, and pages of the same application;
  - Application pages should use the relevant appbridge methods for this, instead of regular links.
    In the future, the appbridge capabilities may expand to make link handling more transparent, so that application developers will be able to use regular `a href` links for this purpose;
- Opening JungleTV-styled alert, confirmation and prompt modals
  - Application pages should use these instead of the deprecated JavaScript methods `alert()`, `prompt()`, etc.
- Calling remote methods on the application's server logic (documented in [a forthcoming section](./rpc.md#remote-methods));
- Emitting events to the server logic, and receiving events from the server logic (documented in [a forthcoming section](./rpc.md#events));
- Parsing Markdown to HTML (merely as a convenience, given the JungleTV SPA already includes code for this. Application pages are free to use their own code for this purpose);
- Automatically synchronizing the application page title such that it becomes visible to the user when relevant, despite the application page being somewhat isolated within an iframe;
- Providing JungleTV-themed UI elements, as described in the next section.

### UI widgets

Through the [Web Components](https://developer.mozilla.org/en-US/docs/Web/API/Web_components) suite of technologies, the appbridge script makes a number of custom HTML elements available for use in application pages.
This way, it is easier to build pages that match the look of the rest of JungleTV, if the application developers so wish.

Scripts executing within the page can interact with these custom elements.
For example, it is possible to attach a click event to a `<jungletv-button>`.
Some elements have [slots](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/slot) that let you add your own markup within the element.
For example, the content of a `<jungletv-button>` is defined within a slot, and the content of the different areas of a `<jungletv-wizard>` is defined using multiple named slots.

The complete list of elements and their functionality is listed in the [appbridge reference section](../reference/appbridge/README.md).

> **Aside**: internally, these UI elements are built using Svelte, just like the JungleTV SPA within which the application pages are displayed.
> These elements are rendered by Svelte, within the context of your application pages, entirely separately from the JungleTV SPA context.
> The appbridge script effectively contains the code for Svelte, albeit without the vast majority of the dependencies of the JungleTV SPA.
> The complexity and dependencies of many JungleTV UI elements limit the selection of elements available through the appbridge.

These components are loaded within [shadow roots](https://developer.mozilla.org/en-US/docs/Web/API/ShadowRoot).
Due to specifics of the Shadow DOM specification around custom fonts in CSS, for these components to be properly styled, they need to also load the CSS bundle for the JungleTV SPA on the DOM of your application page.
This can cause some elements within your pages to be styled differently.
(On the plus side, all of the free [FontAwesome 5 icons](https://fontawesome.com/v5/search?m=free) become available for your page to use, and it should generally be easier for your page to match the usual JungleTV page style.)
The CSS bundle is loaded automatically by the appbridge as soon as, and only if, one of these UI elements is referenced by the page, either through JavaScript e.g. `document.createElement`, or because the element is present in the HTML markup.

The existence of these UI widget implementations does not mean developers are forced to use them.
They are free to invent new UI widgets, reimplement existing ones, and even come up with entirely different UI styles.
For example, for a game themed around monkeys and a jungle, it might make more sense to use banana-themed buttons rather than JungleTV-themed ones.
Keep in mind that, typically, application pages will look a bit awkward if they mix-and-match custom UI styles with those of JungleTV.

## Showing pages as part of the main UI

As previously documented, the [`publishFile()` function](../reference/server/jungletv_pages.md#publishfile) allows pages to be visited through their own URL, where they will appear within the JungleTV SPA (with the JungleTV navigation bar) but still taking over the majority of the screen real estate.
Additionally, that function also unlocks the ability for pages to be displayed on different parts of the main JungleTV UI.

Application pages can be displayed as an additional sidebar tab, akin to the Queue and Chat tabs, with the [`setSidebarTab()` function](../reference/server/jungletv_configuration.md#setsidebartab) of the `jungletv:configuration` module.
Each application can display at most one additional sidebar tab.

Application pages may also be included as chat message attachments.
In such cases, it is necessary to specify how tall the area reserved for the application page should be.
See the [Chat interaction](./chat.md) section for details.

When an application unpublishes a page that is being used in these integrated contexts, or when the application is terminated, typically the UI will revert to a state similar to the one preceding the UI reconfiguration.
In other words, sidebar tabs will disappear and so will the attachment portion of chat messages.
In the case of chat messages, the attached page may reappear if the same application later publishes a page with the same ID.