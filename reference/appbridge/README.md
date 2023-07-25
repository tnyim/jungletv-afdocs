# App bridge

This section contains a reference for the [app bridge script](../../manual/pages.md#client-side-framework-appbridge).
This is a client-side script that enables communication between [application pages](../../manual/pages.md) and the JungleTV client SPA hosting them.
Communication with the server-side application logic also happens through the app bridge.

The app bridge should be embedded in application pages, as the first synchronously loaded script in the page's `<head>` HTML element:

```html
<head>
    <script type="text/javascript" src="**appbridge.js"></script>
    <!-- ... -->
</head>
```

The script assigns a single variable to the global scope, which should be accessed as `window.appbridge`.
This object contains all the methods and properties made available by the app bridge, which are documented on the [app bridge API reference](./api.md).

The app bridge script registers custom HTML elements, documented in the [app bridge custom elements reference](./elements.md), that may be used to easily place JungleTV-themed UI elements within application pages.

Additionally, the app bridge is responsible for:

- Automatically communicating the title of the application page to the host JungleTV SPA, such that it may be shown when and where appropriate.
- Communicating page dimensions to the host JungleTV SPA, such that it can perform adjustments to its DOM when necessary.
- Automatically adding/removing the `dark` class in the application page DOM, so that custom HTML elements and other page elements can assume a dark or light theme in sync with the host page.