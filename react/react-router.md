**Table of Contents**

* [What is React Router](#what-is-react-router)
* [Framework](#framework)
* * [Routing](#routing)
* * * [Route Modules](#route-modules)
* * * [Nested Routes](#nested-routes)
* * * [Root Route](#root-route)
* * * [Layout Routes](#layout-routes)
* * * [Index Routes](#index-routes)
* * * [Route Prefixes](#route-prefixes)
* * * [Dynamic Segments](#dynamic-segments)
* * * [Optional Segements](#optional-segements)
* * * [Splats](#splats)
* * [Route Module](#route-module)
* * * [Component (`default`)](#component-default)
* * * * [Props passed to the component](#props-passed-to-the-component)
* * * * [Using props](#using-props)
* * * [`middleware`](#middleware)
* * * [`clientMiddleware`](#clientmiddleware)
* * * [`loader`](#loader)
* * * [`clientLoader`](#clientloader)
* * * [`action`](#action)
* * * [`clientAction`](#clientaction)
* * * [`ErrorBoundry`](#errorboundry)
* * * [`HydrateFallback`](#hydratefallback)
* * * [`headers`](#headers)
* * * [`handle`](#handle)
* * * [`links`](#links)
* * * [`meta`](#meta)
* * [Navigating](#navigating)
* * * [NavLink](#navlink)
* * * [Link](#link)
* * * [Form](#form)
* * * [`redirect`](#redirect)
* * * [`useNavigate`](#usenavigate)
* * [Pending UI](#pending-ui)
* * * [Global Pending Navigation](#global-pending-navigation)
* * * [Local Pending Navigation](#local-pending-navigation)
* * * [Pending Form Submission](#pending-form-submission)

[<sub>Source</sub>](https://reactrouter.com/start/modes)

---

# What is React Router

React Router is a multi-strategy router for React. There are three primary ways, or "modes", to use it in your app. In this document, we will focus on Framework mode. There are 3 modes in total: Framework mode, Data mode and Declarative mode.

# Framework

Framework Mode wraps Data Mode with a Vite plugin to add the full React Router experience with:

* type-safe `href`
* type-safe Route Module API
* intelligent code splitting
* SPA, SSR, and static rendering strategies
* and more

## Routing

Routes are configured in `app/routes.ts`. Each route has two required parts: a URL pattern to match the URL, and a file path to the route module that defines its behavior.

```tsx
// app/routes.ts

import {
  type RouteConfig,
  route,
} from "@react-router/dev/routes";

export default [
  route("some/path", "./some/file.tsx"),
  // pattern ^           ^ module file
] satisfies RouteConfig;
```

```tsx
import {
  type RouteConfig,
  route,
  index,
  layout,
  prefix,
} from "@react-router/dev/routes";

export default [
  index("./home.tsx"),
  route("about", "./about.tsx"),

  layout("./auth/layout.tsx", [
    route("login", "./auth/login.tsx"),
    route("register", "./auth/register.tsx"),
  ]),

  ...prefix("concerts", [
    index("./concerts/home.tsx"),
    route(":city", "./concerts/city.tsx"),
    route("trending", "./concerts/trending.tsx"),
  ]),
] satisfies RouteConfig;
```

### Route Modules

The files referenced in `routes.ts` define each route's behavior:

```tsx
// app/routes.ts

route("teams/:teamId", "./team.tsx"),
//           route module ^^^^^^^^

```

Example route module:

```tsx
// app/team.tsx

// provides type safety/inference
import type { Route } from "./+types/team";

// provides `loaderData` to the component
export async function loader({ params }: Route.LoaderArgs) {
  let team = await fetchTeam(params.teamId);
  return { name: team.name };
}

// renders after the loader is done
export default function Component({
  loaderData,
}: Route.ComponentProps) {
  return <h1>{loaderData.name}</h1>;
}
```

### Nested Routes

Routes can be nested inside parent routes.

```tsx
import {
  type RouteConfig,
  route,
  index,
} from "@react-router/dev/routes";

export default [
  // parent route
  route("dashboard", "./dashboard.tsx", [
    // child routes
    index("./home.tsx"),
    route("settings", "./settings.tsx"),
  ]),
] satisfies RouteConfig;
```

The path of the parent is automatically included in the child, so this config creates both `"/dashboard"` and `"/dashboard/settings"` URLs.

Child routes are rendered through the `<Outlet/>` in the parent route.

```tsx
// app/dashboard.tsx

import { Outlet } from "react-router";

export default function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>
      {/* will either be home.tsx or settings.tsx */}
      <Outlet />
    </div>
  );
}
```

### Root Route

Every route in routes.ts is nested inside the special `app/root.tsx` module.

### Layout Routes

Using layout, layout routes create new nesting for their children, but they don't add any segments to the URL. It's like the root route but they can be added at any level.

```tsx
// app/routes.ts

import {
  type RouteConfig,
  route,
  layout,
  index,
  prefix,
} from "@react-router/dev/routes";

export default [
  layout("./marketing/layout.tsx", [
    index("./marketing/home.tsx"),
    route("contact", "./marketing/contact.tsx"),
  ]),
  ...prefix("projects", [
    index("./projects/home.tsx"),
    layout("./projects/project-layout.tsx", [
      route(":pid", "./projects/project.tsx"),
      route(":pid/edit", "./projects/edit-project.tsx"),
    ]),
  ]),
] satisfies RouteConfig;
```

> `home.tsx` and `contact.tsx` will be rendered into the `marketing/layout.tsx` outlet without creating any new URL paths
>
> `project.tsx` and `edit-project.tsx` will be rendered into the `projects/project-layout.tsx` outlet at `/projects/:pid` and `/projects/:pid/edit` while `projects/home.tsx` will not.

### Index Routes

```tsx
index(componentFile)
```

Index routes render into their parent's Outlet at their parent's URL (like a default child route).

### Route Prefixes

Using `prefix`, you can add a path prefix to a set of routes without needing to introduce a parent route.

```tsx
import {
  type RouteConfig,
  route,
  layout,
  index,
  prefix,
} from "@react-router/dev/routes";

export default [
  layout("./marketing/layout.tsx", [
    index("./marketing/home.tsx"),
    route("contact", "./marketing/contact.tsx"),
  ]),
  ...prefix("projects", [
    index("./projects/home.tsx"),
    layout("./projects/project-layout.tsx", [
      route(":pid", "./projects/project.tsx"),
      route(":pid/edit", "./projects/edit-project.tsx"),
    ]),
  ]),
] satisfies RouteConfig;
```

Note that this does not introduce a new route into the route tree. Instead, it merely modifies the paths of its children.

For example, these two sets of routes are equivalent:

```tsx
// This usage of `prefix`...
prefix("parent", [
  route("child1", "./child1.tsx"),
  route("child2", "./child2.tsx"),
])

// ...is equivalent to this:
[
  route("parent/child1", "./child1.tsx"),
  route("parent/child2", "./child2.tsx"),
]
```

### Dynamic Segments

If a path segment starts with `:` then it becomes a "dynamic segment". When the route matches the URL, the dynamic segment will be parsed from the URL and provided as `params` to other router APIs.

```tsx
route("teams/:teamId", "./team.tsx"),
```

```tsx
// app/team.tsx

import type { Route } from "./+types/team";

export async function loader({ params }: Route.LoaderArgs) {
  //                           ^? { teamId: string }
}

export default function Component({
  params,
}: Route.ComponentProps) {
  params.teamId;
  //        ^ string
}
```

You can have multiple dynamic segments in one route path:

```tsx
route("c/:categoryId/p/:productId", "./product.tsx"),
```

### Optional Segements

You can make a route segment optional by adding a `?` to the end of the segment.

```tsx
route(":lang?/categories", "./categories.tsx"),
```

You can have optional static segments, too:

```tsx
route("users/:userId/edit?", "./user.tsx");
```

### Splats

Also known as "catchall" and "star" segments. If a route path pattern ends with `/*` then it will match any characters following the `/`, including other `/` characters.

```tsx
route("files/*", "./files.tsx"),
```

```tsx
export async function loader({ params }: Route.LoaderArgs) {
  // params["*"] will contain the remaining URL after files/
}
```

You can destructure the `*`, you just have to assign it a new name. A common name is `splat`:

```tsx
const { "*": splat } = params;
```

## Route Module

The files referenced in `routes.ts` are called Route Modules.

```tsx
route("teams/:teamId", "./team.tsx"),
//           route module ^^^^^^^^
```

### Component (`default`)

The `default` export in a route module defines the component that will render when the route matches.

```tsx
// app/routes/my-route.tsx

export default function MyRouteComponent() {
  return (
    <div>
      <h1>Look ma!</h1>
      <p>
        I'm still using React Router after like 10 years.
      </p>
    </div>
  );
}
```

#### Props passed to the component

When the component is rendered, it is provided the props defined in `Route.ComponentProps` that React Router will automatically generate for you. These props include:

* `loaderData:` The data returned from the `loader` function in this route module
* `actionData:` The data returned from the `action` function in this route module
* `params:` An object containing the route parameters (if any).
* `matches:` An array of all the matches in the current route tree.

You can use these props in place of hooks like `useLoaderData` or `useParams`. This may be preferable because they will be automatically typed correctly for the route.

#### Using props

```tsx
import type { Route } from "./+types/route-name";

export default function MyRouteComponent({
  loaderData,
  actionData,
  params,
  matches,
}: Route.ComponentProps) {
  return (
    <div>
      <h1>Welcome to My Route with Props!</h1>
      <p>Loader Data: {JSON.stringify(loaderData)}</p>
      <p>Action Data: {JSON.stringify(actionData)}</p>
      <p>Route Parameters: {JSON.stringify(params)}</p>
      <p>Matched Routes: {JSON.stringify(matches)}</p>
    </div>
  );
}
```

### `middleware`

Route middleware runs sequentially on the server before and after document and data requests. This gives you a singular place to do things like logging, authentication, and post-processing of responses. The `next` function continues down the chain, and on the leaf route the `next` function executes the loaders/actions for the navigation.

Here's an example middleware to log requests on the server:

```tsx
// root.tsx

async function loggingMiddleware(
  { request, context },
  next,
) {
  console.log(
    `${new Date().toISOString()} ${request.method} ${request.url}`,
  );
  const start = performance.now();
  const response = await next();
  const duration = performance.now() - start;
  console.log(
    `${new Date().toISOString()} Response ${response.status} (${duration}ms)`,
  );
  return response;
}

export const middleware = [loggingMiddleware];
```

Here's an example middleware to check for logged in users and set the user in `context` you can then access from loaders:

```tsx
// routes/_auth.tsx

async function authMiddleware({ request, context }) {
  const session = await getSession(request);
  const userId = session.get("userId");

  if (!userId) {
    throw redirect("/login");
  }

  const user = await getUserById(userId);
  context.set(userContext, user);
}

export const middleware = [authMiddleware];
```

see also:
* [When middleware runs](https://reactrouter.com/how-to/middleware#when-middleware-runs)
* [`middleware` params](https://api.reactrouter.com/v8/types/react-router.MiddlewareFunction.html)
* [Middleware](https://reactrouter.com/how-to/middleware)

### `clientMiddleware`

This is the client-side equivalent of `middleware` and runs in the browser during client navigations. The only difference from server middleware is that client middleware doesn't return Responses because they're not wrapping an HTTP request on the server.

Here's an example middleware to log requests on the client:

```tsx
async function loggingMiddleware(
  { request, context },
  next,
) {
  console.log(
    `${new Date().toISOString()} ${request.method} ${request.url}`,
  );
  const start = performance.now();
  await next(); // 👈 No Response returned
  const duration = performance.now() - start;
  console.log(
    `${new Date().toISOString()} (${duration}ms)`,
  );
  // ✅ No need to return anything
}

export const clientMiddleware = [loggingMiddleware];
```

See also:
* [Client Data](https://reactrouter.com/how-to/client-data)

### `loader`

Route loaders provide data to route components before they are rendered. They are only called on the server when server rendering or during the build with pre-rendering.

see also:
* [`loader` params](https://api.reactrouter.com/v8/interfaces/react-router.LoaderFunctionArgs)

### `clientLoader`

Called only in the browser, route client loaders provide data to route components in addition to, or in place of, route loaders.


```tsx
export async function clientLoader({ serverLoader }) {
  // call the server loader
  const serverData = await serverLoader();
  // And/or fetch data on the client
  const data = getDataFromClient();
  // Return the data to expose through useLoaderData()
  return data;
}
```

Client loaders can participate in initial page load hydration of server rendered pages by setting the `hydrate` property on the function:

```tsx
export async function clientLoader() {
  // ...
}
clientLoader.hydrate = true as const;
```

see also
* [`clientLoader` params](https://api.reactrouter.com/v8/types/react-router.ClientLoaderFunctionArgs)

### `action`

Route actions allow server-side data mutations with automatic revalidation of all loader data on the page when called from `<Form>`, `useFetcher`, and `useSubmit`.

```tsx
// route("/list", "./list.tsx")
import { Form } from "react-router";
import { TodoList } from "~/components/TodoList";

// this data will be loaded after the action completes...
export async function loader() {
  const items = await fakeDb.getItems();
  return { items };
}

// ...so that the list here is updated automatically
export default function Items({ loaderData }) {
  return (
    <div>
      <List items={loaderData.items} />
      <Form method="post" navigate={false} action="/list">
        <input type="text" name="title" />
        <button type="submit">Create Todo</button>
      </Form>
    </div>
  );
}

export async function action({ request }) {
  const data = await request.formData();
  const todo = await fakeDb.addItem({
    title: data.get("title"),
  });
  return { ok: true };
}
```

see also:
* [`action` params](https://api.reactrouter.com/v8/interfaces/react-router.ActionFunctionArgs)

### `clientAction`

```tsx
export async function clientAction({ serverAction }) {
  fakeInvalidateClientSideCache();
  // can still call the server action if needed
  const data = await serverAction();
  return data;
}
```

see also:
* [`clientAction` params](https://api.reactrouter.com/v8/types/react-router.ClientActionFunctionArgs)

### `ErrorBoundry`

When other route module APIs throw, the route module `ErrorBoundary` will render instead of the route component.

```tsx
import {
  isRouteErrorResponse,
  useRouteError,
} from "react-router";

export function ErrorBoundary() {
  const error = useRouteError();

  if (isRouteErrorResponse(error)) {
    return (
      <div>
        <h1>
          {error.status} {error.statusText}
        </h1>
        <p>{error.data}</p>
      </div>
    );
  } else if (error instanceof Error) {
    return (
      <div>
        <h1>Error</h1>
        <p>{error.message}</p>
        <p>The stack trace is:</p>
        <pre>{error.stack}</pre>
      </div>
    );
  } else {
    return <h1>Unknown Error</h1>;
  }
}
```

see also:
* [`useRouteError`](https://reactrouter.com/api/hooks/useRouteError)
* [`isRouteErrorResponse`](https://reactrouter.com/api/utils/isRouteErrorResponse)

### `HydrateFallback`

On initial page load, the route component renders only after the client loader is finished. If exported, a `HydrateFallback` can render immediately in place of the route component.

```tsx
// routes/some-route.tsx

export async function clientLoader() {
  const data = await fakeLoadLocalGameData();
  return data;
}

export function HydrateFallback() {
  return <p>Loading Game...</p>;
}

export default function Component({ loaderData }) {
  return <Game data={loaderData} />;
}
```

### `headers`

The route `headers` function defines the HTTP headers to be sent with the response when server rendering.

```tsx
export function headers() {
  return {
    "X-Stretchy-Pants": "its for fun",
    "Cache-Control": "max-age=300, s-maxage=3600",
  };
}
```

see also:
* [`Headers`](https://developer.mozilla.org/en-US/docs/Web/API/Response/headers)

### `handle`

Route handle allows apps to add anything to a route match in `useMatches` to create abstractions (like breadcrumbs, etc.).

```tsx
export const handle = {
  its: "all yours",
};
```

see also:
* [`useMatches`](https://reactrouter.com/api/hooks/useMatches)

### `links`

Route links define `<link>` elements to be rendered in the document `<head>`.

```tsx
export function links() {
  return [
    {
      rel: "icon",
      href: "/favicon.png",
      type: "image/png",
    },
    {
      rel: "stylesheet",
      href: "https://example.com/some/styles.css",
    },
    {
      rel: "preload",
      href: "/images/banner.jpg",
      as: "image",
    },
  ];
}
```

All routes links will be aggregated and rendered through the `<Links />` component, usually rendered in your app root:

```tsx
import { Links } from "react-router";

export default function Root() {
  return (
    <html>
      <head>
        <Links />
      </head>

      <body />
    </html>
  );
}
```

### `meta`

Route meta defines meta tags to be rendered in the `<Meta />` component, usually placed in the `<head>`.

> Since React 19, using the built-in <meta> element is recommended over the use of the route module's meta export.
>
> Here is an example of how to use it and the `<title>` element:
>
>  ```tsx
>  export default function MyRoute() {
>    return (
>      <div>
>        <title>Very cool app</title>
>        <meta property="og:title" content="Very cool app" />
>        <meta
>          name="description"
>          content="This app is the best"
>        />
>        {/* The rest of your route content... */}
>      </div>
>    );
>  }
> ```

```tsx
export function meta() {
  return [
    { title: "Very cool app" },
    {
      property: "og:title",
      content: "Very cool app",
    },
    {
      name: "description",
      content: "This app is the best",
    },
  ];
}
```

```tsx
import { Meta } from "react-router";

export default function Root() {
  return (
    <html>
      <head>
        <Meta />
      </head>

      <body />
    </html>
  );
}
```

The meta of the last matching route is used, allowing you to override parent routes' meta. It's important to note that the entire meta descriptor array is replaced, not merged. This gives you the flexibility to build your own meta composition logic across pages at different levels.

## Navigating

Users navigate your application with `<Link>`, `<NavLink>`, `<Form>`, `redirect`, and `useNavigate`.

### NavLink

This component is for navigation links that need to render active and pending states.

```tsx
import { NavLink } from "react-router";

export function MyAppNav() {
  return (
    <nav>
      <NavLink to="/" end>
        Home
      </NavLink>
      <NavLink to="/trending" end>
        Trending Concerts
      </NavLink>
      <NavLink to="/concerts">All Concerts</NavLink>
      <NavLink to="/account">Account</NavLink>
    </nav>
  );
}
```

### Link

Use `<Link>` when the link doesn't need active styling:

```tsx
import { Link } from "react-router";

export function LoggedOutMessage() {
  return (
    <p>
      You've been logged out.{" "}
      <Link to="/login">Login again</Link>
    </p>
  );
}
```

### Form

The form component can be used to navigate with `URLSearchParams` provided by the user.

```tsx
<Form action="/search">
  <input type="text" name="q" />
</Form>
```

If the user enters "journey" into the input and submits it, they will navigate to:

```
/search?q=journey
```

Forms with `<Form method="post" />` will also navigate to the action prop but will submit the data as `FormData` instead of `URLSearchParams`. However, it is more common to `useFetcher()` to POST form data.

### `redirect`

Inside of route loaders and actions, you can return a `redirect` to another URL.

```tsx
import { redirect } from "react-router";

export async function loader({ request }) {
  let user = await getUser(request);
  if (!user) {
    return redirect("/login");
  }
  return { userName: user.name };
}
```

It is common to redirect to a new record after it has been created:

```tsx
import { redirect } from "react-router";

export async function action({ request }) {
  let formData = await request.formData();
  let project = await createProject(formData);
  return redirect(`/projects/${project.id}`);
}
```

### `useNavigate`

This hook allows the programmer to navigate the user to a new page without the user interacting. Usage of this hook should be uncommon. It's recommended to use the other APIs in this guide when possible.

Reserve usage of `useNavigate` to situations where the user is not interacting but you need to navigate, for example:

* Logging them out after inactivity
* Timed UIs like quizzes, etc.

```tsx
import { useNavigate } from "react-router";

export function useLogoutAfterInactivity() {
  let navigate = useNavigate();

  useFakeInactivityHook(() => {
    navigate("/logout");
  });
}
```

## Pending UI

When the user navigates to a new route, or submits data to an action, the UI should immediately respond to the user's actions with a pending or optimistic state. Application code is responsible for this.

### Global Pending Navigation

When the user navigates to a new url, the loaders for the next page are awaited before the next page renders. You can get the pending state from `useNavigation`.

```tsx
import { useNavigation } from "react-router";

export default function Root() {
  const navigation = useNavigation();
  const isNavigating = Boolean(navigation.location);

  return (
    <html>
      <body>
        {isNavigating && <GlobalSpinner />}
        <Outlet />
      </body>
    </html>
  );
}
```

### Local Pending Navigation

Pending indicators can also be localized to the link. NavLink's children, className, and style props can be functions that receive the pending state.

```tsx
import { NavLink } from "react-router";

function Navbar() {
  return (
    <nav>
      <NavLink to="/home">
        {({ isPending }) => (
          <span>Home {isPending && <Spinner />}</span>
        )}
      </NavLink>
      <NavLink
        to="/about"
        style={({ isPending }) => ({
          color: isPending ? "gray" : "black",
        })}
      >
        About
      </NavLink>
    </nav>
  );
}
```

### Pending Form Submission

When a form is submitted, the UI should immediately respond to the user's actions with a pending state. This is easiest to do with a fetcher form because it has its own independent state (whereas normal forms cause a global navigation).

```tsx
import { useFetcher } from "react-router";

function NewProjectForm() {
  const fetcher = useFetcher();

  return (
    <fetcher.Form method="post">
      <input type="text" name="title" />
      <button type="submit">
        {fetcher.state !== "idle"
          ? "Submitting..."
          : "Submit"}
      </button>
    </fetcher.Form>
  );
}
```

For non-fetcher form submissions, pending states are available on `useNavigation`.

```tsx
import { useNavigation, Form } from "react-router";

function NewProjectForm() {
  const navigation = useNavigation();

  return (
    <Form method="post" action="/projects/new">
      <input type="text" name="title" />
      <button type="submit">
        {navigation.formAction === "/projects/new"
          ? "Submitting..."
          : "Submit"}
      </button>
    </Form>
  );
}
```
## Rendering Strategies

### Client Side Rendering

Routes are always client side rendered as the user navigates around the app. If you're looking to build a Single Page App, disable server rendering:

```tsx
// react-router.config.ts

import type { Config } from "@react-router/dev/config";

export default {
  ssr: false,
} satisfies Config;
```

### Server Side Rendering

```tsx
import type { Config } from "@react-router/dev/config";

export default {
  ssr: true,
} satisfies Config;
```

Server side rendering requires a deployment that supports it. Though it's a global setting, individual routes can still be statically pre-rendered. Routes can also use client data loading with clientLoader to avoid server rendering/fetching for their portion of the UI.

### Static Pre-rendering

```tsx
import type { Config } from "@react-router/dev/config";

export default {
  // return a list of URLs to prerender at build time
  async prerender() {
    return ["/", "/about", "/contact"];
  },
} satisfies Config;
```

Pre-rendering is a build-time operation that generates static HTML and client navigation data payloads for a list of URLs. This is useful for SEO and performance, especially for deployments without server rendering. When pre-rendering, route module loaders are used to fetch data at build time.

# Notes

## Route Module Function Call Order

1. **Middleware:** Based on `middleware` array insert order
2. **Loader**
3. **Component**