# Application import and export

In order to be executed, JungleTF AF applications must exist as entities in the database of a JungleTV environment.
For editing applications, this is likely their most convenient form, as well - with editing and management commands going through the JungleTV server, it is easier to ensure that applications remain in a valid format.
However, it is possible to export applications out of a JungleTV environment, and into it.

Applications are exported and imported as ZIP files containing the application files.
JAF-specific [file properties](./applications_and_files.md#file-properties) are saved in the extra attributes of the files within the archive.
Other metadata, namely the file history and edit messages, is invariably lost when exporting.
[Application properties](./applications_and_files.md#application-properties) are also not preserved.

Exporting and importing applications enables the following use cases:

- Making backups of applications that are independent of the availability or data integrity of a specific JungleTV environment;
- Restoring applications that were developed in a different environment;
- Exporting all application files in one go for external editing;
- Bulk-import of application files from external sources.

> **Note**: even though certain editors, such as the [VS Code-based web editor](./environments_editors.md#vs-code-based-web-editor), make it easier to drag-and-drop all application files out of an application, the ZIP export option is preferable for creating application backups as it preserves the JAF-specific file properties.

## Exporting applications

JungleTV AF applications can be exported through the [built-in application manager](./environments_editors.md#built-in-application-manager), namely by entering the details view for an application (the screen where you can see the list of files) and pressing the "Export application" button in the "Backup and restore" section.
Shortly, you should be prompted to save a ZIP file to your machine, which contains the latest version of all of the application's files **but not their past versions**.

There is also an "Export application as opaque archive (for emailing)" option, which should be used whenever you need to send the application archive over email, as email providers forbid the attachment of ZIP archives containing JavaScript code.

You should create frequent backups of your applications, especially when using shared development environments such as the [staging lab environment](./environments_editors.md#the-staging-lab-environment).

## Importing applications

JAF applications can be imported also through the [built-in application manager](./environments_editors.md#built-in-application-manager).
An application must already exist - in a way, application ZIP files can be thought of as containing just the contents of an application, but not the "envelope" which is the application itself.

On the details view for an application (the screen where you can see the list of files), the "Backup and restore" section has a file picker where you should select the archive file to import.
You may check "Append only" to only add the files in the archive to the application, instead of the default behavior, which is to replace the entirety of the application files (if any) with those in the archive.
Finally, you should press "Import application" to upload the archive and have its contents imported.
This functionality supports both regular ZIP archives as well as the "opaque archives" mentioned in the previous section.

If the application already contained files whose names matched those in the archive, the files from the archive are added as new versions of those files (regardless of the "Append only" setting).
This is relevant in the context of file rollback features, to be offered in the future.

If the ZIP archive contains directories, the directory structure is ignored and all files within the archive are imported recursively.
If there are multiple files with the same name within different directories, there is no rule as to which file "wins."
All the conflicting files will be present as (superseded) versions of the JAF application file with the name.

When importing applications, the JAF looks for the file properties in the ZIP file extra attributes, in the format it used when exporting.
If the metadata is not present (e.g. because the ZIP file was produced with other software), the MIME type of the files will be guessed (using [this](https://github.com/gabriel-vasile/mimetype) package) and all files will be set as internal.
You should verify and correct these file properties after importing.