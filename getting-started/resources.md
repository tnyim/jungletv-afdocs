# Development resources

We tried to minimize the number of tools needed to develop JungleTV AF applications; developers don't need to install anything on their machines besides a web browser.
This reduces the time commitment needed to begin playing around with the JAF, and makes application development less intimidating to novice developers.
However, there is nothing stopping experienced developers from using a mix of external tools and more advanced practices to develop applications - in which case the ability to [import](../manual/import_export.md#importing-applications) applications as archives may be useful.

This section will shine light on the resources designed specifically to help with JAF application development, including a guide on accessing a hosted development environment, marking the start of the hands-on portion of this guide.

## Prerequisites

Development of JungleTV AF applications requires some proficiency with the technologies and standards embraced by the framework, as this is not a guide to teach JavaScript or programming in general.
Prospective developers should have:

- Knowledge about what JungleTV is and how it functions from the perspective of the users;
- Some programming experience, either with JavaScript specifically, or with a number of other imperative programming languages so that they can easily learn JavaScript as they go;
  - If you are unsure if you have sufficient experience, try giving the rest of the guide a chance and see whether you lose track of what is happening!
- Some experience with HTML and CSS - even though a JAF application does not necessarily need to contain web pages, they are the recommended way to build an user interface;
- Basic technical awareness of how the Web works and the technologies and concepts commonly associated with it: what is a web server, what is a URL, what is a domain name, what's an API, what's JSON, etc.;
- A desktop, laptop or other computer that allows mouse and keyboard interaction and the execution of a full-featured modern web browser - as the application development tools are not really well suited for small screens or the exclusive use of touchscreens;
- An internet connection;
- No fear of asking questions and "poking around to find out".

In terms of the primary language used in JAF applications, JavaScript, and general guidance on web development, including on how to use HTML and CSS, we recommend the [MDN Web Docs](https://developer.mozilla.org/).
Throughout the manual, you will see some links to them.

## Lab environment

The easiest way to develop applications for JungleTV is to use JungleTV itself.
Well, not exactly the production version of the website, but a dedicated _lab environment_ that is available for everyone to use, at [staging.jungletv.live](https://staging.jungletv.live).
This special environment has changes specifically to make application development possible, including the ability for anyone to have staff privileges and thus have access to the application editor that is built into JungleTV.
Its database is completely separate from that of the production version of the website; in general, it is a wholly separate website from that at jungletv.live, it just happens to be powered by the same software.

Let's sign into this environment with staff privileges, and take what is probably your first look at the tools available to JungleTV staff!

> **Note**: The staging lab environment is a public resource destined for mature and civil people.
Its users must abide by the [staging environment etiquette](../manual/environments_editors.md#staging-environment-etiquette).
Keeping this environment running, with a low barrier to entry, is only possible if everyone makes an effort to keep it easy to moderate.
We are unlikely to give second chances to those who do not heed this notion.

1. Visit staging.jungletv.live and head to the ["Earn rewards" page](https://staging.jungletv.live/rewards/address);
2. On the first field, enter one of _your_ Banano addresses, that will be used to identify you on this environment - just like what happens on regular JungleTV;
3. On the second field, enter `earlyafaccess`;
   - This is what will give your session elevated privileges; if you were signing in with normal privileges (e.g. to test an application), you'd leave this field blank.
4. Complete the authentication process as normal;
   - Don't forget that you can later head to the [Rewards page](https://staging.jungletv.live/rewards) and press the button to edit your profile and set a nickname, so you can be more easily identified in this environment.
5. You can now visit the [moderation dashboard](https://staging.jungletv.live/moderate), only accessible by directly entering its URL (there are no links to it on the JungleTV SPA itself).

Most of what is on the moderation dashboard is related to, well, community moderation and configuring different aspects of the service.
Right now, we are mostly interested in the JungleTV AF capabilities, so let's head over to the application editor.

## The built-in editor

The JungleTV SPA contains an interface for application development.
It is not the most friendly editor for writing code, but it is the only one providing access to some features (such as application creation, import and export and application property editing), so you should become familiar with it.

To access this editor, press the "Manage applications" link on the moderation dashboard (or access [this link](https://staging.jungletv.live/moderate/applications) directly).
[Later in this guide](./example_hello.md#the-application-editor), we'll go through the creation and execution of a simple application, and give more details on how to use this editor.

## The VS Code-based web editor

For writing more than a few lines of code or managing more than a few application files, we recommend using a web editor that the JungleTV team has put together, based on Visual Studio code and [github1s](https://github.com/conwnet/github1s).
It comes with a JungleTV AF helper extension preinstalled, you are able to install more extensions as long as they are compatible with VS Code Web, and you can otherwise customize it to your liking.

This editor is available at [editor.jungletv.live](https://editor.jungletv.live).
Setting it up is not required to continue following this guide - we'll remind you to do it at a later point.
Instructions are available in the [relevant section of the manual](../manual/environments_editors.md#using-the-editor).

## External tools

As part of developing JungleTV AF applications, you'll almost certainly eventually need to use tools like a color picker, an image editor, or a spellchecker.
Naturally, our editors won't be the end-all-be-all of application development - although, by installing extensions on the VS Code-based editor, you could probably get pretty close!

It should go without saying, but you are free to use any external tool to help with the development of JAF applications.
Even in terms of code editing, you can use a different editor of your choice and import the files into a JungleTV environment afterwards.
This likely won't have as short of a feedback loop compared to using one of the two designated editors, as you'll have to keep reuploading files (even for this, we recommend the VS Code-based web editor - as you'll be able to simply drag-and-drop files into the explorer view!).

## The manual and reference

Beyond this getting started guide, this documentation website continues on to provide a [complete manual](../manual) and a [reference section](../reference).
Once you are done with this guide and wish to give application development a serious chance, we strongly recommend reading through the manual.

## Discussion channel

The BananoLabs Discord server [has a channel dedicated to JungleTV](https://discord.gg/YYdJ3Ztf3t).
It is the place to ask questions and give feedback about the JungleTV Application Framework.
Don't be afraid to use it - we are aware that the JAF can be intimidating, and the documentation can always be improved.