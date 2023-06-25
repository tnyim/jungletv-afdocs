# Best practices for code structure and file organization

When developing software projects of moderate to high complexity, organization is paramount.
JungleTV AF applications are no exception.
While there are different techniques and methodologies that can be applied, the consistent application of some patterns or conventions - _any_ ones - is often the most important aspect.
You are free to decide what works best for you and your team, but the JAF may impose some changes to how you usually structure your code.

In this section, we will give some tips and suggestions on how to structure the code and files of JAF applications, even in the absence of features such as folders for application files.

## Structuring code

JavaScript and TypeScript code can be split among multiple files.
When code in those files is written in a particular way, they become **modules**.
Modules can _import_ code (functions, or any other type of values) that is _exported_ by other modules.
This allows you to split code across files.

> **Note**: When writing JavaScript code for server-side execution in the JAF, you should use the CommonJS syntax to define and import modules, which is the older syntax first [introduced by Node.js](https://nodejs.org/api/modules.html#modules-commonjs-modules).
> In CommonJS modules, values (which may be an object containing multiple other values) are exported by assigning them to `module.exports`.
> The value assigned can then be referenced in other files, through the use of `require("./filename.js")`.
>
> When writing TypeScript code for server-side execution in the JAF, you may use the more modern syntax introduced in ES6, where values are typically exported by prefixing them with the keyword `export`, and other modules may import these through a syntax similar to `import { someFunction, someValue } from "./filename";`.
>
> When writing code for in-browser execution, you should also be able to use the ES6 modules syntax, given that it is supported by all browsers for which JungleTV is designed.
> [See more about JavaScript modules on MDN web docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules).

The exact way you will split code among modules in your JAF applications will depend on their specifics, and simpler applications can likely get away with keeping all the code in a single file (or two files, one for the server script and another for a public script to run in the users' browsers).

Constants and static configuration parameters for your applications are frequently good candidates for living in a separate file/module.
On that note, keep in mind that you can parse and import the contents of JSON application files (which must have the correct `application/json` MIME type) into your server scripts, using the `require` syntax exclusively.

For TypeScript code, types that are used by both server and client code - for example, the types of the parameters and return values of RPC methods - can live in files that are imported (using `import type`) by both server and client scripts.
This will help ensure that both the server and client logic "speak" the same language.

Unfortunately, you are likely to run into trouble when trying to write JavaScript modules that work on both the server and the browser, as the server uses a module style (CommonJS) that is different from that supported by the browsers (ES6).
A future JAF version may include functionality to help with this use case, when writing TypeScript.

On a last JAF-specific note on modules, while you can `require` a module at any time throughout code execution (i.e. not necessarily only when an application begins running), you should be aware that module loading is synchronous and may pause execution for a brief but significant amount of time.
Particularly when importing TypeScript modules for the first time within an application's execution cycle, the JAF has to transpile the code to JavaScript, which could take up to a couple seconds, or even longer in the case of very complex code.
If this happens within the "hot path" of your application, for example when computing the response to an RPC call, it could result in noticeable "hiccups" or sluggishness to the users.
As a rule of thumb, it is best to keep module imports near the top of each file, outside of functions.

### Radically different approaches

You may opt to write your code entirely outside of the JAF and deploy your applications as single files, created by a bundler such as [webpack](https://webpack.js.org/) or [rollup.js](https://rollupjs.org/).
You may even want to use a language or framework that is neither pure JavaScript nor TypeScript, but still compiles to the former.
We do not officially support these configurations, but in principle there is nothing _in the current JAF implementation_ that would prevent them from working.
Note that if you use processes that make it significantly harder to understand the logic within the final files at a high level (e.g. terser, minification, complex frameworks such as React or Svelte), you may be asked to provide the original sources as part of the [review process](./review_deployment.md).

## Organizing files

The main limitation of the JAF in terms of file organization, compared to most other systems, is the lack of support for folders.
This is unlikely to be a problem for projects of low to moderate complexity, but for projects with a large number of assets, it may become a point of friction.
Another big difference is that the JAF has a file property specifically for the MIME type of each file.
Here are some ideas on how to avoid descending into chaos:

- **Take advantage of the alphabetical sort order**: the built-in application manager and other [editors](./environments_editors.md#editors) typically display files sorted from A to Z.
  You may therefore group files through the use of prefixes.
- **Take advantage of image sprites**: if your application pages make use of many small images, it may be an interesting idea to condense all of these into a single image and use [CSS image sprites](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_images/Implementing_image_sprites_in_CSS) and other similar techniques to visually split the condensed image into its various parts.
- **Keep using file extensions**: even though the JAF, and web browsers, largely consider the MIME type property instead of the file extension, for interoperability with other systems and for your own reference you'll likely want to keep extensions on your file names.
  The JAF encourages this, by expecting files with an extension (`main.js`, `main.ts`) for the application entry point.