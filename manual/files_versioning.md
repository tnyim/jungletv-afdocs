# Applications, files and versioning

Other than a bit of metadata, JungleTV AF applications are defined by little more than a collection of files that contain their code and assets.

When an application is launched, the JAF runtime tries to find one of two files within the application: `main.js` and `main.ts`, in this order.
If neither can be found, the application fails to launch.
Otherwise, the application JavaScript environment is brought up, the event loop begins running and the first found file is executed within that environment.
The code within this file can reference other files belonging to that application, as needed.

## Application files

Application files exist inside of what can be considered a very limited filesystem, specific to each application and wholly independent and unreachable by other JAF applications.

This virtual filesystem does not support directories; it also does not support files whose names start with a `*` (asterisk), or files with empty names.
File names are encoded in UTF-8 and can have up to 128 code points ("characters").
In theory, there are no other limitations on file names.

Files have two important metadata fields, or properties, associated with them:

- **MIME type**: also known as Media Types or Content Types, [MIME types](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types) define the nature and format of a file.
  Much like browsers primarily use MIME types, rather than file extensions, to determine how to process files, the JungleTV AF **completely ignores file extensions** and exclusively uses MIME types to determine how to process each application file, and to judge their fitness for certain uses.
  It is therefore extremely important that application files have their MIME types set correctly.
- **Public/Internal**: files can be marked as Public, which will make them available for download by anyone, on the web, over HTTPS, **while the application is running**.
  Files are served with a `Content-Type` header matching their MIME type, reinforcing the importance of this file property.
  Public files can be referenced/"imported" by [application pages](./pages.md).
  You should never count on a public file, whose existence was disclosed at one point, to be "forgotten" just because the application was stopped or because the file was set as internal later on - users can download files, caches exist, etc.
  Scripts intended to be run in the users' browsers must be marked as public (as otherwise they can't be fetched), while scripts intended to run on the server are typically kept as internal (but there's nothing that enforces this, and you can have files containing code that is able to run on either environment).

Application files can be uploaded and edited either through the [application editors](./editors.md).
When creating or editing a file, you will usually be prompted to enter an **edit message**.
This is for your own reference and the reference of other developers, and should be useful mainly when looking to revert to older versions of a given file.
This is similar to e.g. commit messages in Git - even though the version control system within the JAF is much more limited than Git, as we'll see next.
Each file version is also associated with the JungleTV user that created it.

For an application to be executable, it must contain at least one of the following files - even if they are empty (which could make sense, if you just want to use the application to serve static files):
- `main.js` with MIME type `application/javascript` and containing valid JavaScript code
- `main.ts` with MIME type `application/typescript` and containing valid TypeScript code

Application files that aren't executable code or JSON can't be directly read by application code, but this may change in a future JAF version.
Application files can't be manipulated by their application or a different JAF application, and there are presently no plans to change this limitation.
This is to encourage a separation between an application's code and static assets, and the application's data.

In terms of implementation, application files exist as rows on a JungleTV database table dedicated to application files.
Because files are versioned - as described in the next section - each file can actually correspond to multiple rows in the database.
## File versioning

The simple filesystem used for applications has an equally simple built-in file versioning system.

Every time a file is saved, a new version is created, which can be identified by the date and time of the modification and - for human reference - the **edit message** and JungleTV user associated with that version.
Files continue to exist in the database even when they are deleted.
In other words, the deletion of a file merely creates a new version for it, where the file is marked as deleted.

There is presently no way to recover old versions of application files without contacting JungleTV staff, but this is planned for a future milestone.
All file versions are already being kept together with their edit messages.

File versions are used internally by the framework to help with proxy/browser cache busting - ensuring that the public files that get served to [application pages](./pages.md) are always the ones for the right version of the application.
This means you should not need to worry about cache busting tricks when referencing application files within your pages.

Application files are only completely lost when an application is deleted; in other words, all edits are "soft" other than application deletion, which is always a "hard" delete.

## Application properties

Let's now discuss the few non-file bits that make up an application.

Applications have an ID, encoded in UTF-8.
Other than a maximum length of 36 code points, there are presently no restrictions on application IDs - but this may change at a later point.
We encourage using only lowercase letters and digits in your application IDs, and preferably do not start with a digit, or in regular expression terms, `[a-z][a-z0-9]*`.
Application IDs can't be changed other than by duplicating an application, which will cause all application edit history to be lost as it will not be transferred to the newly created application.

Application IDs are visible to users, for example, in the URLs for [application pages](./pages.md).

Applications also have the latest edit message, edit author and modification date associated with them.
As when editing a file, you will be prompted for an edit message when changing application properties.
These configurable metadata fields associated with each JAF application are:

- **Allow editing**: whether the application files can be edited.
  This is mainly to prevent accidental destruction of projects considered "final" or a "reference."
- **Allow launching**: whether the application can be launched.
  The purpose of this bit is to prevent accidental launching of applications that are not ready to be executed (e.g. because they are fundamentally incomplete, or because they have a serious unresolved bug that could result in e.g. data loss).
- **Run on server start-up**: whether to automatically launch the application when the server starts.
  This may make sense for certain types of applications that are meant as more permanent or long-running additions to JungleTV.

## Application versioning

Applications have versions, which are simply the modification date of the application.
Therefore, every single change that is made to an application - either to its files or its properties - results in a new application version.
This effectively means that there is one application version for each version of its files, but there may be application versions without a corresponding file edit.

When an application is launched, a version must be specified.
Running applications are "frozen in time," specifically, the files that are considered throughout execution are those of the application version that was launched.
(Or in even more precise terms: the file versions considered are the most recent ones, for each individual file, that do not exceed the launched application version.)

Presently, the JungleTV AF only allows the most recent version of an application to be launched.
In practice, this means that files are stuck as they were at the time the application was launched - even if they've later been edited or deleted, or if new files have been added.
