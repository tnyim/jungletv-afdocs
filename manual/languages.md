# Language support

As detailed in the [Architecture overview](./architecture.md), the JungleTV AF largely consists of a [JavaScript](https://developer.mozilla.org/en-US/docs/Web/javascript) engine running within the JungleTV server, and therefore it is no surprise that it supports this language, whose name was formally standardized as ECMAScript (JavaScript is a trademark registered by Oracle in some countries).
This means that the same language can be used for both server-side code and code that runs in the browser, as part of [application pages](./pages.md).

Additionally, the JAF also natively supports [TypeScript](https://www.typescriptlang.org/), a language developed by Microsoft that is designed to transpile to, and is formally a superset of, JavaScript.
Its main purpose is to add an optional [static type system](https://en.wikipedia.org/wiki/Type_system#STATIC) to JavaScript, turning it into a much more type-safe language.
In simple terms, TypeScript makes it easier to write code whose behavior matches what the program authors intended, making it significantly easier to write complex, large-scale JavaScript applications.
A static type system makes it easier for code editors, compilers and associated tooling to identify certain types of mistakes in the code before it even executes, as well as provide better information to programmers, such as rich autocomplete options, when writing code.

## JavaScript support

The extent of the JavaScript support in the JungleTV AF differs between code that runs on the server and code that runs on the users' browsers.

### On the server
The JungleTV server uses [Goja](https://github.com/dop251/goja) to execute JavaScript.
Its support for language features lags somewhat behind that of more robust and popular engines such as V8, and because it is largely just an interpreter, its speed is also orders of magnitude behind JavaScript engines that make use of [Just-in-Time compilation](https://en.wikipedia.org/wiki/Just-in-time_compilation) and other even more advanced execution techniques.
In practice, we believe you won't run into any limitations arising from Goja.

Goja supports all of the ES5 standard and most of the ES6 features.
If compiling/transpiling to JavaScript from other languages, ES2017 is a good target to use.

Goja module support largely matches the CommonJS standard, so you must use the `require` function to import code, and `module.exports` to export code from a module ("file").
Similar to what is done in Node.js, you can also load JSON application files in your JavaScript contexts using `require` - just ensure they have the correct `application/json` MIME type.

### On the browser
Client scripts are executed by the users' browsers, meaning there is a wider variety of engines and language support to consider.
In practice, the JungleTV client application is only expected to run on browsers supporting ES2021 features; more generally, the JungleTV developers only target "mainstream browsers" that are up-to-date, i.e. not really more than a year behind their latest versions.
"Mainstream browsers" consist on Firefox, Chromium/Blink-based browsers, and iOS Safari.

Because your JavaScript code will run within [application pages](./pages.md) which themselves run as part of the JungleTV client application, you should not bother with support for browsers older than what the client application itself supports.
This rule of thumb applies to other client-side languages and features as well, namely CSS and HTML, image format support, etc.

## TypeScript support

In the JungleTV Application Framework, TypeScript is a first-class citizen.
You will find that you can write TypeScript code and it will be executed seamlessly, **even when referencing scripts in browser contexts**: the JAF will transparently take care of transpiling TypeScript to JavaScript whenever necessary.
It is important that the [MIME type](./applications_and_files.md#application-properties) of the files containing TypeScript be set to `application/typescript`.

The current TypeScript version used by the JAF is 4.9.3.
We expect to update this version as the dependencies used by the JungleTV server allow.

Even though the JAF mostly ignores file extensions, instead looking at the MIME types set for each file, you should still give JavaScript and TypeScript files the `.js` and `.ts` extensions, respectively, to avoid confusion and for proper IDE support, namely when using the [VS Code-based web editor](./environments_editors.md#vs-code-based-web-editor), whose language tooling is not aware of the JAF MIME type property.

While the JAF will compile your TypeScript, it does not perform any type checking.
Code will still be transpiled and executed even if it does not pass TypeScript validation.
This is largely because of how the TypeScript compiler within the JAF processes each file individually and asynchronously, ignoring types imported from other files or types of JAF native modules.
However, you will be able to check for type errors when using the [VS Code-based web editor](./environments_editors.md#vs-code-based-web-editor), which performs holistic analysis of all files within each opened application and takes into consideration the types of the [native modules](../reference/server/README.md) and [appbridge](../reference/appbridge/README.md).

### TypeScript in server scripts

You can write some or all of your code in `application/typescript` files.
TypeScript files will be transpiled to JavaScript - so Goja can execute them - as soon as they are imported by the server-side code.
The main application entry point may be written in TypeScript: in this case, in addition to having the `application/typescript` MIME type, the main file should be called `main.ts` instead of `main.js`.

You can mix and match TypeScript and JavaScript files, importing modules written in TypeScript from JavaScript server scripts and vice-versa.

The TypeScript code emitted by the internal compilation process will include a code map to ensure that any line number references in exceptions match those of the original TypeScript code.

Using TypeScript in server scripts will let you use certain JavaScript features not natively supported by Goja, as the TypeScript compiler will take care of introducing polyfills and other code changes to transparently make your code compatible with the limitations of Goja, in accordance with the TypeScript compiler options passed by the JAF when transpiling.
Notably, when writing TypeScript you should use the more modern import syntax (e.g. `import { someFunction } from "somemodule";`) that is otherwise not supported when writing JavaScript.

### TypeScript in browser contexts

Assuming that you adhere to the best practice of not having inline scripts in your HTML markup, you can write the entirety of the code to be run on the users' machines in TypeScript, with the JAF taking care of transparently compiling the code to JavaScript with the appropriate compiler options.

Within the HTML markup of your [application pages](./pages.md), you just need to reference files as if they were JavaScript scripts:

```html
<script type="text/javascript" src="some_script.ts">
```

Assuming the file is correctly marked as having the `application/typescript` MIME type, then, in the process of serving these scripts to clients, the server will compile them to JavaScript.
In practice, you never have to write or even see JavaScript code when writing JAF application code.

## Other languages

In theory, the JAF is able to support server code in any language that can be compiled/transpiled to the JavaScript standard supported by Goja.
Beware that if using such languages, or tools like minifiers or tersers, you should provide the original source code as part of the [review process](./review_deployment.md).
The JAF server does not support WebAssembly (WASM).

On the browser, within [application pages](./pages.md), you are more or less free to use all current web standards, including [WebAssembly](https://developer.mozilla.org/en-US/docs/WebAssembly).
Beware: the use of WebAssembly could cause some antivirus/security solutions to immediately flag the website as malware, so ensure you have a valid use case before turning to this technology.
WASM binaries can be included as part of the application files, and marked as public so they can be served to browsers.
The source code for any WASM binaries will likely be requested during application review.

On the browser, you will find that the lowest common denominator in terms of feature support, and the browser most likely to require extra testing and adjustments to your code, is going to be iOS Safari (which, for the time being, is the engine used for all iOS browsers regardless of their "brand").