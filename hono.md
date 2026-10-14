**Table of Contents**

* [What is Hono](#what-is-hono)
* [Developer Experience](#developer-experience)
* [Basics](#basics)
* * [Return JSON](#return-json)
* * [Request and Response](#request-and-response)
* * [Return HTML](#return-html)
* * [Return raw Response](#return-raw-response)
* * [Using Middleware](#using-middleware)
* * [Adapter](#adapter)
* * [Change port number](#change-port-number)
* * [Serve static files](#serve-static-files)
* * * [`rewriteRequestPath`](#rewriterequestpath)
* * * [`mimes`](#mimes)
* * * [`onFound`](#onfound)
* * * [`onNotFound`](#onnotfound)
* * * [`precompressed`](#precompressed)
* * [Testing](#testing)

[<sub>Source</sub>](https://hono.dev/docs/getting-started/basic)

--- 

# What is Hono?

Hono is a small, simple, and ultrafast web framework built on Web Standards. It works on any JavaScript runtime: Cloudflare Workers, Fastly Compute, Deno, Bun, Vercel, Netlify, AWS Lambda, Lambda@Edge, and Node.js.

**Quick start:**

```bash
# use any package manager
bun create hono@latest
```

```ts
// server index.ts
import { Hono } from 'hono'
const app = new Hono()

app.get('/', (c) => c.text('Hono!'))

export default app
```

# Developer Experience

Hono provides a delightful "Developer Experience".

Easy access to Request/Response thanks to the `Context` object. Moreover, Hono is written in TypeScript. Hono has **"Types"**.

And, the Validator and Hono Client `hc` enable the RPC mode. In RPC mode, you can use your favorite validator such as `Zod` and easily share server-side API specs with the client and build type-safe applications.

# Basics

You can write code in TypeScript with the Cloudflare Workers development tool "Wrangler", Deno, Bun, or others without being aware of transpiling.

Write your first application with Hono in `src/index.ts`. The example below is a starter Hono application.

The `import` and the final `export default` parts may vary from runtime to runtime, but all of the application code will run the same code everywhere.

```ts
import { Hono } from 'hono'

const app = new Hono()

app.get('/', (c) => {
  return c.text('Hello Hono!')
})

export default app
```

## Return JSON

Returning JSON is also easy. The following is an example of handling a GET Request to `/api/hello` and returning an `application/json` Response.

```ts
app.get('/api/hello', (c) => {
  return c.json({
    ok: true,
    message: 'Hello Hono!',
  })
})
```

## Request and Response

Getting a path parameter, URL query value, and appending a Response header is written as follows.

```ts
app.get('/posts/:id', (c) => {
  const page = c.req.query('page')
  const id = c.req.param('id')
  c.header('X-Message', 'Hi!')
  return c.text(`You want to see ${page} of ${id}`)
})
```

We can easily handle POST, PUT, and DELETE not only GET.

```ts
app.post('/posts', (c) => c.text('Created!', 201))
app.delete('/posts/:id', (c) =>
  c.text(`${c.req.param('id')} is deleted!`)
)
```

## Return HTML

You can write HTML with the html Helper or using JSX syntax. If you want to use JSX, rename the file to `src/index.tsx` and configure it (check with each runtime as it is different). Below is an example using JSX.

```tsx
const View = () => {
  return (
    <html>
      <body>
        <h1>Hello Hono!</h1>
      </body>
    </html>
  )
}

app.get('/page', (c) => {
  return c.html(<View />)
})
```

## Return raw Response

```ts
app.get('/', () => {
  return new Response('Good morning!')
})
```

## Using Middleware

Middleware can do the hard work for you. For example, add in Basic Authentication.

```ts
import { basicAuth } from 'hono/basic-auth'

// ...

app.use(
  '/admin/*',
  basicAuth({
    username: 'admin',
    password: 'secret',
  })
)

app.get('/admin', (c) => {
  return c.text('You are authorized!')
})
```

There are useful built-in middleware including Bearer and authentication using JWT, CORS and ETag. Hono also provides third-party middleware using external libraries such as GraphQL Server and Firebase Auth. And, you can make your own middleware.

## Adapter

There are Adapters for platform-dependent functions, e.g., handling static files or WebSocket. They are published as separate packages. For example, to handle WebSocket in Cloudflare Workers, install and import `@hono/cloudflare-workers`.

```ts
import { upgradeWebSocket } from '@hono/cloudflare-workers'

app.get(
  '/ws',
  upgradeWebSocket((c) => {
    // ...
  })
)
```

## Change port number

You can specify the port number with exporting the `port`.

```ts
import { Hono } from 'hono'

const app = new Hono()
app.get('/', (c) => c.text('Hello Bun!'))

export default { 
  port: 3000, 
  fetch: app.fetch, 
} 
```

## Serve static files

To serve static files, use `serveStatic` from the `@hono/bun` package.

```bash
bun add @hono/bun
```

```ts
import { serveStatic } from '@hono/bun'

const app = new Hono()

app.use('/static/*', serveStatic({ root: './' }))
app.use('/favicon.ico', serveStatic({ path: './favicon.ico' }))
app.get('/', (c) => c.text('You can access: /static/hello.txt'))
app.get('*', serveStatic({ path: './static/fallback.txt' }))
```

For the above code, it will work well with the following directory structure.

```
./
├── favicon.ico
├── src
└── static
    ├── demo
    │   └── index.html
    ├── fallback.txt
    ├── hello.txt
    └── images
        └── dinotocat.png
```

### `rewriteRequestPath`

If you want to map `http://localhost:3000/static/*` to `./statics`, you can use the `rewriteRequestPath` option:

```ts
app.get(
  '/static/*',
  serveStatic({
    root: './',
    rewriteRequestPath: (path) =>
      path.replace(/^\/static/, '/statics'),
  })
)
```

### `mimes`

You can add MIME types with `mimes`:

```ts
app.get(
  '/static/*',
  serveStatic({
    mimes: {
      m3u8: 'application/vnd.apple.mpegurl',
      ts: 'video/mp2t',
    },
  })
)
```

### `onFound`

You can specify handling when the requested file is found with `onFound`:

```ts
app.get(
  '/static/*',
  serveStatic({
    // ...
    onFound: (_path, c) => {
      c.header('Cache-Control', `public, immutable, max-age=31536000`)
    },
  })
)
```

### `onNotFound`

You can specify handling when the requested file is not found with `onNotFound`:

```ts
app.get(
  '/static/*',
  serveStatic({
    onNotFound: (path, c) => {
      console.log(`${path} is not found, you access ${c.req.path}`)
    },
  })
)
```

### `precompressed`

The `precompressed` option checks if files with extensions like `.br` or `.gz` are available and serves them based on the `Accept-Encoding` header. It prioritizes Brotli, then Zstd, and Gzip. If none are available, it serves the original file.

```ts
app.get(
  '/static/*',
  serveStatic({
    precompressed: true,
  })
)
```

## Testing

You can use `bun:test` for testing on Bun.

```ts
import { describe, expect, it } from 'bun:test'
import app from '.'

describe('My first test', () => {
  it('Should return 200 Response', async () => {
    const req = new Request('http://localhost/')
    const res = await app.fetch(req)
    expect(res.status).toBe(200)
  })
})
```

```bash
bun test index.test.ts
```