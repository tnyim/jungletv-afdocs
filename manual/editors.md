# Editors and development environments

The JungleTV Application Framework largely makes use of common technology standards and tools - like the JavaScript programming language.
As described in the [import and export section](./import_export.md), in theory one can even develop JAF applications entirely independently from JungleTV, as long as one can create text files and compress them in a ZIP file.
Still, JAF applications are much easier to develop when using the tools built specifically for the framework.
We have developed multiple options for more comfortably editing, debugging and managing applications.

## Editors

Editors interact with a running JungleTV environment to edit [application files and properties](./applications_and_files.md) and launch, stop and monitor JAF applications.
Most of the development effort of a JAF application will likely involve interacting with one of these editors.

### Built-in application manager

This is the primary way of interacting with the JungleTV AF backoffice.
When it comes to JAF-specific aspects, the application manager and editor built into the JungleTV client SPA is the most powerful and direct of all available editing options.
It lets one view and control each application and their files, including their JAF-specific properties, and is the only supported interface for [importing and exporting applications](./import_export.md).

The built-in web editor can be accessed at the `/moderate/applications` path of an executing JungleTV environment.

A notable functionality of the application manager is the ability to see the console of running applications, to which applications can send debug messages and interactively run JavaScript within each application.

Unfortunately, this manager does not offer the best code editing experience, offering just a rather basic code editor (based on [CodeMirror](https://codemirror.net/)) with limited quality-of-life features.
For example, it lacks rich autocomplete for JAF native modules and your own source code, and does not perform any syntax or type checking.
It is meant as a last-resort option for quick edits.

Intensive file management is also not very comfortable in this editor, which has limited drag-and-drop capabilities, cannot perform bulk operations on files, and does not support the most common keyboard shortcuts for file management.

### VS Code extension

For a code editing experience much superior to what the built-in editor could hope to achieve - at least, without turning into a much larger project than JungleTV itself - a [Visual Studio Code](https://code.visualstudio.com/) extension was developed, allowing for a much more integrated application development experience.

This extension presents JungleTV application files as a virtual filesystem within VS Code.
This means there are some limitations compared to editing local files, but the file management story is generally better, in comparison with the application manager built into JungleTV.
For example, dragging and dropping files into the VS Code file explorer, and out of it, is possible, as is selecting multiple files.
Certain JAF-specific aspects (like setting file MIME types) are not natively supported within VS Code, but the extension adds context menu options to manage these.

The extension also presents itself as a "debugger."
While "true" step debugging of JAF applications is not possible at this moment, what the extension does is present the application console as the debug console, allowing it to be used within VS Code, including the evaluation of JavaScript expressions.

The extension is not published in any VS Code extension marketplace, but its [source code is available on GitHub](https://github.com/tnyim/jungletv-vsc/).

Due to limitations in the built-in support VS Code has for JavaScript and TypeScript, when running as a regular desktop install (i.e. not web based), there is unfortunately limited code completion and type checking features, compared to what would normally be available when editing regular files on your computer.
Because of this, we don't actually recommend installing the extension in regular VS Code - even though all of the features provided by the extension itself will work perfectly.
Read on for the recommended approach.

### VS Code-based web editor

To work around the limitations of VS Code's built in support for the TypeScript language when editing files on virtual file systems, and to lower the barrier to entry for JAF application development even further, a custom distribution of VS Code, designed for use in web browsers, in was developed.

This editor is available online, at **https://editor.jungletv.live**.
It works better in browsers based on the Blink browser engine (e.g. Chromium, Chrome), as that is what VS Code is designed for.

> **Aside**: This custom distribution of VS Code works around the TypeScript language support woes, by disabling the built-in support and using [this extension](https://github.com/volarjs/vscode-typescript-web) instead.
> Unfortunately, this extension does not work when VS Code is running in a non-web context, so it is not a suitable workaround for the problems experienced when using VS Code proper.
> The source code for the custom distribution of VS Code is available [on GitHub](https://github.com/tnyim/jungletv-github1s).
> It is based on [github1s](https://github.com/conwnet/github1s).

The beauty of this editor is that it offers an application development experience comparable to that of a desktop environment (or actually superior, due to the aforementioned workaround), even on devices that are limited in their support for desktop software, such as tablets and Chromebooks.
Like with the [built-in application manager](#built-in-application-manager), developers are not required to install any specific software in order to begin developing JAF applications.

**[[TODO] complete]**

## Development environments

**[[TODO] Placeholder]**