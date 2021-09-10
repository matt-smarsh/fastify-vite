
# Using React

<b>fastify-vite</b>'s 
[base React 17 starter boilerplate](...) is based on the [official React SSR example][ssr-react] from Vite's [playground][playground]. The differences start with `server.js`, where the [raw original _Express_-based example][react-server.js] can be replaced with the following Fastify server initialization boilerplate:

[react-server.js]: https://github.com/vitejs/vite/blob/main/packages/playground/ssr-react/server.js
[ssr-react]: https://github.com/vitejs/vite/tree/main/packages/playground/ssr-react
[playground]: https://github.com/vitejs/vite/tree/main/packages/playground

<table class="infotable">
<tr>
<td style="width: 20%">
<div class="language-"><pre><code>
├─ entry/
│  ├─ client.jsx
│  └─ server.jsx
├─ views/
│  ├─ index.jsx
│  └─ about.jsx
├─ index.html
├─ base.jsx
├─ routes.js
├─ main.js
└─ <b style="color: #ec6f2d">server.js</b>
</code></pre></div>
</td>
<td>

```js

const fastify = require('fastify')
const fastifyVite = require('fastify-vite')
const fastifyViteReact = require('fastify-vite-react')

async function main () {
  const app = fastify()
  await app.register(fastifyVite, {
    root: __dirname,
    renderer: fastifyViteReact,
    build: process.argv.includes('build'),
  })
  return fastify
}

if (require.main === module) {
  main().then((app) => {
    app.listen(3000, (err, address) => {
      if (err) {
        console.error(err)
        process.exit(1)
      }
      console.log(`Server listening on ${address}`)
    })
  })
}

module.exports = main
```

</td>
</tr>
</table>

You may notice instantly `main()` doesn't call `app.listen()` directly. This is an established <b>_idiom_</b> to facilitate <b>testing</b>, that is, having a function that returns the preconfigured Fastify instance.

Notice how we also pass in `build` flag, based on the presence of a `build` command line argument. If `build` is `true`, running the following command would then <b>trigger the Vite build</b> for your app instead of booting the server, as it'll force `process.exit()` when the build is done:

<code style="font-size: 1.2em">$ node server.js build</code>

::: tip
This mimics the behavior of [vite build](), calling Vite's internal `build()` function and will take into consideration options defined in a `vite.config.js` file or provided via the `vite` plugin option.
:::

## Entry Points 

The next differences from Vite's official React SSR example are the <b>server</b> and <b>client</b> entry points.

For the <b>server</b> entry point, instead of providing only a `render` function, with <b>fastify-vite</b> you can also provide a `routes` array. The `render` function itself should be created with the factory function provided by <b>fastify-vite-react</b>, `createRenderFunction()`, which will automate things like [client hydration]() and add support for [route hooks](), [payloads]() and [isomorphic data fetching]().

[server-entry-point]: https://github.com/vitejs/vite/blob/main/packages/playground/ssr-react/src/entry-server.js 

<table class="infotable">
<tr>
<td style="width: 20%">
<div class="language-"><pre><code>
├─ <b style="color: #ec6f2d">entry/</b>
│  ├─ client.jsx
│  └─ <b style="color: #ec6f2d">server.jsx</b>
├─ views/
│  ├─ index.jsx
│  └─ about.jsx
├─ index.html
├─ base.jsx
├─ routes.js
├─ main.js
└─ server.js
</code></pre></div>
</td>
<td>

```js

import { createApp } from '../main'
import { createRenderFunction } from 'fastify-vite-react/server'
import routes from '../routes'

export default {
  routes,
  render: createRenderFunction(createApp),
}
```

</td>
</tr>
</table>

For the <b>client</b> entry point, things are nearly exact the same as [the original example][client-entry-point], the only addition being the `hydrate()` import and call seen in the snippet below. 

[client-entry-point]: https://github.com/vitejs/vite/blob/main/packages/playground/ssr-react/src/entry-client.js 

<table class="infotable">
<tr>
<td style="width: 20%">
<div class="language-"><pre><code>
├─ <b style="color: #ec6f2d">entry/</b>
│  ├─ <b style="color: #ec6f2d">client.jsx</b>
│  └─ server.jsx
├─ views/
│  ├─ index.jsx
│  └─ about.jsx
├─ index.html
├─ base.jsx
├─ routes.js
├─ main.js
└─ server.js
</code></pre></div>
</td>
<td>

```js

import ReactDOM from 'react-dom'
import { ContextProvider, hydrate } from 'fastify-vite-react/client'
import { createApp } from '../main'

const { App, router: Router } = createApp()

ReactDOM.hydrate(
  <Router>
    <ContextProvider context={hydrate()}>
      {App()}
    </ContextProvider>
  </Router>,
  document.getElementById('app'),
)
```

</td>
</tr>
</table>

This will pick up values serialized in `window` during SSR (like `window.__NUXT__`) and make sure they're available through `useHydration()`, <b>fastify-vite</b>'s unified helper for dealing with isomorphic data. See more in <b>[Client Hydration]()</b>, <b>[Route Payloads]()</b> and <b>[Isomorphic Data]()</b>. 

## Routing Setup


You can set routes directly from JSX view files, just export `path`:

<table class="infotable">
<tr>
<td style="width: 20%">
<div class="language-"><pre><code>
├─ entry/
│  ├─ client.js
│  └─ server.js
├─ views/
│  ├─ <b style="color: #ec6f2d">index.jsx</b>
│  └─ about.jsx
├─ index.html
├─ base.jsx
├─ routes.js
├─ main.js
└─ server.js
</code></pre></div>
</td>
<td>

```jsx

import { Link } from 'react-router-dom'

export const path = '/'

export default function Index () {
  return (
    <>
      <h1>Index Page</h1>
      <p>Go to <Link to="/about">/about</Link></p>
    </>
  )
}
```

</td>
</tr>
</table>

As long as you also use the `loadRoutes()` helper from <b>fastify-vite-vue/app</b>:

<table class="infotable">
<tr>
<td style="width: 20%">
<div class="language-"><pre><code>
├─ entry/
│  ├─ client.js
│  └─ server.js
├─ views/
│  ├─ index.jsx
│  └─ about.jsx
├─ index.html
├─ base.jsx
├─ <b style="color: #ec6f2d">routes.js</b>
├─ main.js
└─ server.js
</code></pre></div>
</td>
<td>

```js

import { loadRoutes } from 'fastify-vite-vue/app'

export default loadRoutes(import.meta.globEager('./views/*.jsx'))
```

</td>
</tr>
</table>

The following snippet is equivalent to the one above:


<table class="infotable">
<tr>
<td style="width: 20%">
<div class="language-"><pre><code>
├─ entry/
│  ├─ client.jsx
│  └─ server.jsx
├─ views/
│  ├─ index.jsx
│  └─ about.jsx
├─ index.html
├─ base.jsx
├─ <b style="color: #ec6f2d">routes.js</b>
├─ main.js
└─ server.js
</code></pre></div>
</td>
<td>

```js

import Index from './views/index.jsx'
import About from './views/about.jsx'

export default [
  {
    path: '/',
    component: Index,
  },
  {
    path: '/about',
    component: About,
  },
]
```

</td>
</tr>
</table>


Similarly to the way `createRenderFunction()` works, providing a `routes` array in your server entry export is what ensures you can have individual Fastify [route hooks](), [payloads]() and [isomorphic data]() functions for each of your [React Router][react-router] routes. When these are exported directly from your view files, `loadRoutes()` ensures they're collected. 

<b>fastify-vite</b> [will use]() this array to automatically <b>register one individual route</b> for them while applying any hooks and data functions provided.

[vue-router]: https://reactrouter.com/

::: tip
<b>If you don't export</b> `routes`, you have to tell Fastify what routes you want rendering your SSR application:

```js
fastify.vite.get('/*')
```

And in this case, any <b>_hooks_</b> or <b>_data functions_</b> exported directly from your Vue files <b>would be ignored</b>.
:::