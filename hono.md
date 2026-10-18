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
* [Routing](#routing)
* * [Basic](#basic)
* * [Path Parameter](#path-parameter)
* * [Optional Parameter](#optional-parameter)
* * [Regexp](#regexp)
* * [Including Slashes](#including-slashes)
* * [Grouping](#grouping)
* * [Base Path](#base-path)
* * [Routing Priority](#routing-priority)
* [Context](#context)
* * [req](#req)
* * [status()](#status)
* * [header()](#header)
* * [body()](#body)
* * [text()](#text)
* * [json()](#json)
* * [html()](#html)
* * [notFound()](#notfound)
* * [redirect()](#redirect)
* * [res](#res)
* * [set() / get()](#set--get)
* * [var](#var)
* * [error](#error)
* [Middlewares](#middlewares)
* * [Basic Authentication](#basic-authentication)
* * [Bearer Authentication](#bearer-authentication)
* * [JWT Authentication](#jwt-authentication)
* * [CORS](#cors)
* * * [Using With Vite](#using-with-vite)
* * [Pretty JSON](#pretty-json)

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

# Routing

## Basic

```ts
// HTTP Methods
app.get('/', (c) => c.text('GET /'))
app.post('/', (c) => c.text('POST /'))
app.put('/', (c) => c.text('PUT /'))
app.delete('/', (c) => c.text('DELETE /'))
app.query('/', (c) => c.text('QUERY /'))

// Wildcard
app.get('/wild/*/card', (c) => {
  return c.text('GET /wild/*/card')
})

// Any HTTP methods
app.all('/hello', (c) => c.text('Any Method /hello'))

// Custom HTTP method
app.on('PURGE', '/cache', (c) => c.text('PURGE Method /cache'))

// Multiple Method
app.on(['PUT', 'DELETE'], '/post', (c) =>
  c.text('PUT or DELETE /post')
)

// Multiple Paths
app.on('GET', ['/hello', '/ja/hello', '/en/hello'], (c) =>
  c.text('Hello')
)
```

## Path Parameter

```ts
app.get('/user/:name', async (c) => {
  const name = c.req.param('name')
  // ...
})
```

or all parameters at once:

```ts
app.get('/posts/:id/comment/:comment_id', async (c) => {
  const { id, comment_id } = c.req.param()
  // ...
})
```

## Optional Parameter 

```ts
// Will match `/api/animal` and `/api/animal/:type`
app.get('/api/animal/:type?', (c) => c.text('Animal!'))
```

## Regexp

```ts
app.get('/post/:date{[0-9]+}/:title{[a-z]+}', async (c) => {
  const { date, title } = c.req.param()
  // ...
})
```

## Including slashes

```ts
app.get('/posts/:filename{.+\\.png}', async (c) => {
  //...
})
```

## Grouping

You can group the routes with the Hono instance and add them to the main app with the route method.

```ts
const book = new Hono()

book.get('/', (c) => c.text('List Books')) // GET /book
book.get('/:id', (c) => {
  // GET /book/:id
  const id = c.req.param('id')
  return c.text('Get Book: ' + id)
})
book.post('/', (c) => c.text('Create Book')) // POST /book

const app = new Hono()
app.route('/book', book)
```

You can also group multiple instances while keeping base.

```ts
const book = new Hono()
book.get('/book', (c) => c.text('List Books')) // GET /book
book.post('/book', (c) => c.text('Create Book')) // POST /book

const user = new Hono().basePath('/user')
user.get('/', (c) => c.text('List Users')) // GET /user
user.post('/', (c) => c.text('Create User')) // POST /user

const app = new Hono()
app.route('/', book) // Handle /book
app.route('/', user) // Handle /user
```

## Base Path

You can specify the base path.

```ts
const api = new Hono().basePath('/api')
api.get('/book', (c) => c.text('List Books')) // GET /api/book
```

## Routing priority

Handlers or middleware will be executed in registration order.

```ts
app.get('/book/a', (c) => c.text('a')) // a
app.get('/book/:slug', (c) => c.text('common')) // common
```

```
GET /book/a ---> `a`
GET /book/b ---> `common`
```

When a handler is executed, the process will be stopped.

```ts
app.get('*', (c) => c.text('common')) // common
app.get('/foo', (c) => c.text('foo')) // foo
```

```
GET /foo ---> `common` // foo will not be dispatched
```

If you have the middleware that you want to execute, write the code above the handler. And If you want to have a "fallback" handler, write the code below the other handler.

```ts
app.use(logger()) // middleware . 1
app.get('/foo', (c) => c.text('foo')) // foo . 2
app.get('*', (c) => c.text('fallback')) // fallback . 3
```

# Context

The `Context` object is instantiated for each request and kept until the response is returned. You can put values in it, set headers and a status code you want to return, and access HonoRequest and Response objects.

## req

`req` is an instance of HonoRequest

```ts
app.get('/hello', (c) => {
  const userAgent = c.req.header('User-Agent')
  // ...
})
```

## status()

You can set an HTTP status code with `c.status()`. The default is `200`. You don't have to use `c.status()` if the code is `200`.

```ts
app.post('/posts', (c) => {
  // Set HTTP status code
  c.status(201)
  return c.text('Your post is created!')
})
```

## header()

You can set HTTP Headers for the response.

```ts
app.get('/', (c) => {
  // Set headers
  c.header('X-Message', 'My custom message')
  return c.text('Hello!')
})
```

## body()

Return an HTTP response.

> **Note:** When returning text or HTML, it is recommended to use `c.text()` or `c.html()`.

```ts
app.get('/welcome', (c) => {
  c.header('Content-Type', 'text/plain')
  // Return the response body
  return c.body('Thank you for coming')
})
```

```ts
app.get('/welcome', (c) => {
  return c.body('Thank you for coming', 201, {
    'X-Message': 'Hello!',
    'Content-Type': 'text/plain',
  })
})
```

```ts
new Response('Thank you for coming', {
  status: 201,
  headers: {
    'X-Message': 'Hello!',
    'Content-Type': 'text/plain',
  },
})
```

## text()

Render text as `Content-Type: text/plain`.

```ts
app.get('/say', (c) => {
  return c.text('Hello!')
})
```

## json()

Render JSON as `Content-Type: application/json`.

```ts
app.get('/api', (c) => {
  return c.json({ message: 'Hello!' })
})
```

## html()

Render HTML as `Content-Type: text/html`.

```ts
app.get('/', (c) => {
  return c.html('<h1>Hello! Hono!</h1>')
})
```

## notFound()

Return a `Not Found` Response. You can customize it with `app.notFound()`.

```ts
app.get('/notfound', (c) => {
  return c.notFound()
})
```

## redirect()

Redirect, default status code is `302`.

```ts
app.get('/redirect', (c) => {
  return c.redirect('/')
})
app.get('/redirect-permanently', (c) => {
  return c.redirect('/', 301)
})
```

## res

You can access the Response object that will be returned.

```ts
// Response object
app.use('/', async (c, next) => {
  await next()
  c.res.headers.append('X-Debug', 'Debug message')
})
```

## set() / get()

Get and set arbitrary key-value pairs, with a lifetime of the current request. This allows passing specific values between middleware or from middleware to route handlers.

```ts
app.use(async (c, next) => {
  c.set('message', 'Hono is cool!!')
  await next()
})

app.get('/', (c) => {
  const message = c.get('message')
  return c.text(`The message is "${message}"`)
})
```

Pass the `Variables` as Generics to the constructor of `Hono` to make it type-safe.

```ts
type Variables = {
  message: string
}

const app = new Hono<{ Variables: Variables }>()
```

The value of `c.set` / `c.get` are retained only within the same request. They cannot be shared or persisted across different requests.

## var

You can also access the value of a variable with `c.var`.

```ts
const result = c.var.client.oneMethod()
```

If you want to create the middleware which provides a custom method, write like the following:

```ts
type Env = {
  Variables: {
    echo: (str: string) => string
  }
}

const app = new Hono()

const echoMiddleware = createMiddleware<Env>(async (c, next) => {
  c.set('echo', (str) => str)
  await next()
})

app.get('/echo', echoMiddleware, (c) => {
  return c.text(c.var.echo('Hello!'))
})
```

If you want to use the middleware in multiple handlers, you can use `app.use()`. Then, you have to pass the `Env` as Generics to the constructor of `Hono` to make it type-safe.

```ts
const app = new Hono<Env>()

app.use(echoMiddleware)

app.get('/echo', (c) => {
  return c.text(c.var.echo('Hello!'))
})
```

## error

If the Handler throws an error, the error object is placed in `c.error`. You can access it in your middleware.

```ts
app.use(async (c, next) => {
  await next()
  if (c.error) {
    // do something...
  }
})
```

# Middlewares

For more middlewares, check [Source](https://hono.dev/docs/)

## Basic Authentication

This middleware can apply Basic authentication to a specified path. Implementing Basic authentication with Cloudflare Workers or other platforms is more complicated than it seems, but with this middleware, it's a breeze.

```ts
import { Hono } from 'hono'
import { basicAuth } from 'hono/basic-auth'

const app = new Hono()

app.use(
  '/auth/*',
  basicAuth({
    username: 'hono',
    password: 'acoolproject',
  })
)

app.get('/auth/page', (c) => {
  return c.text('You are authorized')
})
```

To restrict to a specific route + method:

```ts
const app = new Hono()

app.get('/auth/page', (c) => {
  return c.text('Viewing page')
})

app.delete(
  '/auth/page',
  basicAuth({ username: 'hono', password: 'acoolproject' }),
  (c) => {
    return c.text('Page deleted')
  }
)
```

If you want to verify the user by yourself, specify the `verifyUser` option; returning `true` means it is accepted.

```ts
const app = new Hono()

app.use(
  basicAuth({
    verifyUser: (username, password, c) => {
      return (
        username === 'dynamic-user' && password === 'hono-password'
      )
    },
  })
)
```

**Options:**

* #required username: `string`
* #required password: `string`
* #optional realm: `string`
* * The domain name of the realm, as part of the returned WWW-Authenticate challenge header. The default is "Secure Area".
* #optional hashFunction: `Function`
* #optional verifyUser: `(username: string, password: string, c: Context) => boolean | Promise<boolean>`
* #optional invalidUserMessage: `string | object | MessageFunction`
* * MessageFunction is `(c: Context) => string | object | Promise<string | object>`. The custom message if the user is invalid.
* #optional onAuthSuccess: `(c: Context, username: string) => void | Promise<void>`
* * A callback function invoked after successful authentication. This allows you to set context variables or perform side effects without re-parsing the Authorization header.

```ts
app.use(
  '/auth/*',
  basicAuth({
    username: 'hono',
    password: 'acoolproject',
    onAuthSuccess: (c, username) => {
      c.set('username', username)
    },
  })
)

app.get('/auth/page', (c) => {
  const username = c.get('username')
  return c.text(`Hello, ${username}!`)
})
```

* #optional ...users: `{ username: string, password: string }[]`

## Bearer Authentication

The Bearer Auth Middleware provides authentication by verifying an API token in the Request header. The HTTP clients accessing the endpoint will add the `Authorization` header with `Bearer {token}` as the header value.

> Your token must match the regex /[A-Za-z0-9._~+/-]+=*/, otherwise a 400 error will be returned. Notably, this regex accommodates both URL-safe Base64- and standard Base64-encoded JWTs. This middleware does not require the bearer token to be a JWT, just that it matches the above regex.

```ts
import { Hono } from 'hono'
import { bearerAuth } from 'hono/bearer-auth'

const app = new Hono()

const token = 'honoiscool'

app.use('/api/*', bearerAuth({ token }))

app.get('/api/page', (c) => {
  return c.json({ message: 'You are authorized' })
})
```

To restrict to a specific route + method:

```ts
const app = new Hono()

const token = 'honoiscool'

app.get('/api/page', (c) => {
  return c.json({ message: 'Read posts' })
})

app.post('/api/page', bearerAuth({ token }), (c) => {
  return c.json({ message: 'Created post!' }, 201)
})
```

To implement multiple tokens (E.g., any valid token can read but create/update/delete are restricted to a privileged token):

```ts
const app = new Hono()

const readToken = 'read'
const privilegedToken = 'read+write'
const privilegedMethods = ['POST', 'PUT', 'PATCH', 'DELETE']

app.on('GET', '/api/page/*', async (c, next) => {
  // List of valid tokens
  const bearer = bearerAuth({ token: [readToken, privilegedToken] })
  return bearer(c, next)
})
app.on(privilegedMethods, '/api/page/*', async (c, next) => {
  // Single valid privileged token
  const bearer = bearerAuth({ token: privilegedToken })
  return bearer(c, next)
})

// Define handlers for GET, POST, etc.
```

If you want to verify the value of the token yourself, specify the `verifyToken` option; returning `true` means it is accepted.

```ts
const app = new Hono()

app.use(
  '/auth-verify-token/*',
  bearerAuth({
    verifyToken: async (token, c) => {
      return token === 'dynamic-token'
    },
  })
)
```

## JWT Authentication

The JWT Auth Middleware provides authentication by verifying the token with JWT. The middleware will check for an `Authorization` header if the `cookie` option is not set. You can customize the header name using the `headerName` option.

> The Authorization header sent from the client must have a specified scheme.
>
>Example: `Bearer my.token.value` or `Basic my.token.value`

```ts
import { Hono } from 'hono'
import { jwt } from 'hono/jwt'
import type { JwtVariables } from 'hono/jwt'

// Specify the variable types to infer the `c.get('jwtPayload')`:
type Variables = JwtVariables

const app = new Hono<{ Variables: Variables }>()

app.use(
  '/auth/*',
  jwt({
    secret: 'it-is-very-secret',
    alg: 'HS256',
  })
)

app.get('/auth/page', (c) => {
  return c.text('You are authorized')
})
```

Get payload:

```ts
const app = new Hono()

app.use(
  '/auth/*',
  jwt({
    secret: 'it-is-very-secret',
    alg: 'HS256',
    verification: {
      iss: 'my-trusted-issuer',
      aud: 'my-api',
    },
  })
)

app.get('/auth/page', (c) => {
  const payload = c.get('jwtPayload')
  return c.json(payload) // eg: { "sub": "1234567890", "name": "John Doe", "iat": 1516239022, "iss": "my-trusted-issuer" }
})
```

## CORS

There are many use cases of Cloudflare Workers as Web APIs and calling them from external front-end application. For them we have to implement CORS, let's do this with middleware as well.

```ts
import { Hono } from 'hono'
import { cors } from 'hono/cors'

const app = new Hono()

// CORS should be called before the route
app.use('/api/*', cors())
app.use(
  '/api2/*',
  cors({
    origin: 'http://example.com',
    allowHeaders: ['X-Custom-Header', 'Upgrade-Insecure-Requests'],
    allowMethods: ['POST', 'GET', 'OPTIONS'],
    exposeHeaders: ['Content-Length', 'X-Kuma-Revision'],
    maxAge: 600,
    credentials: true,
  })
)

app.all('/api/abc', (c) => {
  return c.json({ success: true })
})
app.all('/api2/abc', (c) => {
  return c.json({ success: true })
})
```

Multiple origins:

```ts
app.use(
  '/api3/*',
  cors({
    origin: ['https://example.com', 'https://example.org'],
  })
)

// Or you can use "function"
app.use(
  '/api4/*',
  cors({
    // `c` is a `Context` object
    origin: (origin, c) => {
      return origin.endsWith('.example.com')
        ? origin
        : 'http://example.com'
    },
  })
)
```

### Using with Vite 

When using Hono with Vite, you should disable Vite's built-in CORS feature by setting `server.cors` to `false` in your `vite.config.ts`. This prevents conflicts with Hono's CORS middleware.

```ts
// vite.config.ts
import { cloudflare } from '@cloudflare/vite-plugin'
import { defineConfig } from 'vite'

export default defineConfig({
  server: {
    cors: false, // disable Vite's built-in CORS setting
  },
  plugins: [cloudflare()],
})
```

## Pretty JSON

Pretty JSON middleware enables "JSON pretty print" for JSON response body. Adding `?pretty` to url query param, the JSON strings are prettified.

```// GET /
{"project":{"name":"Hono","repository":"https://github.com/honojs/hono"}}
```

will be:

```
// GET /?pretty
{
  "project": {
    "name": "Hono",
    "repository": "https://github.com/honojs/hono"
  }
}
```

```ts
import { Hono } from 'hono'
import { prettyJSON } from 'hono/pretty-json'

const app = new Hono()

app.use(prettyJSON()) // With options: prettyJSON({ space: 4 })
app.get('/', (c) => {
  return c.json({ message: 'Hono!' })
})
```