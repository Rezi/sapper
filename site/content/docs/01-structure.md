---
title: Sapper app structure
---

This section is a reference for the curious. We recommend you play around with the project template first, and come back here when you've got a feel for how things fit together.

If you take a look inside the [sapper-template](https://github.com/sveltejs/sapper-template) repo, you'll see some files that Sapper expects to find:

```bash
├ package.json
├ src
│ ├ routes
│ │ ├ # your routes here
│ │ ├ _error.svelte
│ │ └ index.svelte
│ ├ client.js
│ ├ server.js
│ ├ service-worker.js
│ └ template.html
├ static
│ ├ # your files here
└ rollup.config.js / webpack.config.js
```

When you first run Sapper, it will create an additional `__sapper__` directory containing generated files.

You'll notice a few extra files and a `cypress` directory which relates to [testing](docs#Testing) — we don't need to worry about those right now.

> You *can* create these files from scratch, but it's much better to use the template. See [getting started](docs#Getting_started) for instructions on how to easily clone it


### package.json

Your package.json contains your app's dependencies and defines a number of scripts:

* `npm run dev` — start the app in development mode, and watch source files for changes
* `npm run build` — build the app in production mode
* `npm run export` — bake out a static version, if applicable (see [exporting](docs#Exporting))
* `npm start` — start the app in production mode after you've built it
* `npm test` — run the tests (see [testing](docs#Testing))


### src

This contains the three *entry points* for your app — `src/client.js`, `src/server.js` and (optionally) `src/service-worker.js` — along with a `src/template.html` file.

#### src/client.js

This *must* import, and call, the `start` function from the generated `@sapper/app` module:

```js
import * as sapper from '@sapper/app';

sapper.start({
	target: document.querySelector('#sapper')
});
```

In many cases, that's the entirety of your entry module, though you can do as much or as little here as you wish. See the [client API](docs#Client_API) section for more information on functions you can import.


#### src/server.js

This is a normal Express (or [Polka](https://github.com/lukeed/polka), etc) app, with three requirements:

* it should serve the contents of the `static` folder, using for example [sirv](https://github.com/lukeed/sirv)
* it should call `app.use(sapper.middleware())` at the end, where `sapper` is imported from `@sapper/server`
* it must listen on `process.env.PORT`

Beyond that, you can write the server however you like.


#### src/service-worker.js

Service workers act as proxy servers that give you fine-grained control over how to respond to network requests. For example, when the browser requests `/goats.jpg`, the service worker can respond with a file it previously cached, or it can pass the request on to the server, or it could even respond with something completely different, such as a picture of llamas.

Among other things, this makes it possible to build applications that work offline.

Because every app needs a slightly different service worker (sometimes it's appropriate to always serve from the cache, sometimes that should only be a last resort in case of no connectivity), Sapper doesn't attempt to control the service worker. Instead, you write the logic in `service-worker.js`. You can import any of the following from `@sapper/service-worker`:

* `files` — an array of files found in the `static` directory
* `shell` — the client-side JavaScript generated by the bundler (Rollup or webpack)
* `routes` — an array of `{ pattern: RegExp }` objects you can use to determine whether a Sapper-controlled page is being requested
* `timestamp` — the time the service worker was generated (useful for generating unique cache names)


#### src/template.html

This file is a template for responses from the server. Sapper will inject content that replaces the following tags:

* `%sapper.base%` — a `<base>` element (see [base URLs](docs#Base_URLs))
* `%sapper.styles%` — critical CSS for the page being requested
* `%sapper.head%` — HTML representing page-specific `<head>` contents, like `<title>`
* `%sapper.html%` — HTML representing the body of the page being rendered
* `%sapper.scripts%` — script tags for the client-side app


### src/routes

This is the meat of your app — the pages and server routes. See the section on [routing](docs#Routing) for the juicy details.


### static

This is a place to put any files that your app uses — fonts, images and so on. For example `static/favicon.png` will be served as `/favicon.png`.

Sapper doesn't serve these files — you'd typically use [sirv](https://github.com/lukeed/sirv) or [serve-static](https://github.com/expressjs/serve-static) for that — but it will read the contents of the `static` folder so that you can easily generate a cache manifest for offline support (see [service-worker.js](docs#templates-service-worker-js)).


### rollup.config.js / webpack.config.js

Sapper can use [Rollup](https://rollupjs.org/) or [webpack](https://webpack.js.org/) to bundle your app. You probably won't need to change the config, but if you want to (for example to add new loaders or plugins), you can.