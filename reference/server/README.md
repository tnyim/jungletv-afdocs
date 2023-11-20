# Server modules reference

This section contains a reference for all the APIs available to code running on the server.

For modules that are not imported by default, the reference pages show an example of how to import them.
If you are using TypeScript on your server scripts (see [Language support](/manual/languages.md)) note that, while the shown way will work, the recommended way to import modules _in TypeScript code only_ is to use the more modern ES6 import syntax.
So for example, to import the `jungletv:chat` module, use `import chat from "jungletv:chat"` instead of `require("jungletv:chat")`.

<!--  keep this in sync with _sidebar.md -->

- [`jungletv:chat`](/reference/server/jungletv_chat.md)
- [`jungletv:configuration`](/reference/server/jungletv_configuration.md)
- [`jungletv:db`](/reference/server/jungletv_db.md)
- [`jungletv:keyvalue`](/reference/server/jungletv_keyvalue.md)
- [`jungletv:pages`](/reference/server/jungletv_pages.md)
- [`jungletv:points`](/reference/server/jungletv_points.md)
- [`jungletv:queue`](/reference/server/jungletv_queue.md)
- [`jungletv:rpc`](/reference/server/jungletv_rpc.md)
- [`node:console`](/reference/server/node_console.md)
- [`node:process`](/reference/server/node_process.md)
- [`node:util`](/reference/server/node_util.md)