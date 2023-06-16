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

Unfortunately, this manager does not offer the best code editing experience, offering just a rather basic code editor (based on [CodeMirror](https://codemirror.net/)) with limited quality-of-life features.
For example, it lacks rich autocomplete for JAF native modules and your own source code, and does not perform any syntax or type checking.
It is meant as a last-resort option for quick edits.

Intensive file management is also not very comfortable in this editor, which has limited drag-and-drop capabilities, cannot perform bulk operations on files, and does not support the most common keyboard shortcuts for file management.

### VS Code extension

**[[TODO] Placeholder]**

### VS Code-based web editor

**[[TODO] Placeholder]**

## Development environments

**[[TODO] Placeholder]**