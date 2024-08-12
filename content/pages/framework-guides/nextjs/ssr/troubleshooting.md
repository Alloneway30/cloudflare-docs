---
pcx_content_type: troubleshooting
title: Troubleshooting
meta:
  title: Troubleshooting | Full-stack (SSR) | Next.js apps
---

# Troubleshooting

Learn more about troubleshooting issues with your Full-stack (SSR) Next.js apps using Cloudflare.

## Edge runtime

You must configure all server-side routes in your Next.js project as [Edge runtime](https://nextjs.org/docs/app/building-your-application/rendering/edge-and-nodejs-runtimes) routes, by adding the following to each route::

```js
export const runtime = "edge";
```

{{<Aside type="note">}}
If you are still using the Next.js [Pages router](https://nextjs.org/docs/pages), for page routes, you must use `'experimental-edge'` instead of `'edge'`.
{{</Aside>}}

---

## App router

### Not found

Next.js generates a `not-found` route for your application under the hood during the build process. In some circumstances, Next.js can detect that the route requires server-side logic (particularly if computation is being performed in the root layout component) and Next.js automatically creates a [Node.js runtime serverless function](https://nextjs.org/docs/app/building-your-application/rendering/edge-and-nodejs-runtimes) that is not compatible with Cloudflare Pages.

To prevent this, you can provide a custom `not-found` route that explicitly uses the edge runtime:

```ts
---
filename: (src/)app/not-found.(jsx|tsx)
---

export const runtime = 'edge'

export default async function NotFound() {
    // ...
    return (
        // ...
    )
}
```

### `generateStaticParams`

When you use [static site generation (SSG)](https://nextjs.org/docs/pages/building-your-application/rendering/static-site-generation) in the [`/app` directory](https://nextjs.org/docs/getting-started/project-structure) and also use the [`generateStaticParams`](https://nextjs.org/docs/app/api-reference/functions/generate-static-params) function, Next.js tries to handle requests for non statically generated routes automatically, and creates a [Node.js runtime serverless function](https://nextjs.org/docs/app/building-your-application/rendering/edge-and-nodejs-runtimes) that is not compatible with Cloudflare Pages.

You can opt out of this behavior by setting [`dynamicParams`](https://nextjs.org/docs/app/api-reference/file-conventions/route-segment-config#dynamicparams) to `false`:

```diff
---
filename: app/my-example-page/[slug]/page.jsx
---
+ export const dynamicParams = false

// ...
```

### Top-level `getRequestContext`

You must call `getRequestContext` within the function that handles your route — it cannot be called in global scope.

Don't do this:

```js
---
filename: app/api/myvar/route.js
highlight: [5]
---
import { getRequestContext } from '@cloudflare/next-on-pages'

export const runtime = 'edge'

const myVariable = getRequestContext().env.MY_VARIABLE

export async function GET(request) {
  return new Response(myVariable)
}
```

Instead, do this:

```js
---
filename: app/api/myvar/route.js
highlight: [6]
---
import { getRequestContext } from '@cloudflare/next-on-pages'

export const runtime = 'edge'

export async function GET(request) {
  const myVariable = getRequestContext().env.MY_VARIABLE
  return new Response(myVariable)
}
```

---

## Pages router

### `getStaticPaths`

When you use [static site generation (SSG)](https://nextjs.org/docs/pages/building-your-application/rendering/static-site-generation) in the [`/pages` directory](https://nextjs.org/docs/getting-started/project-structure) and also use the [`getStaticPaths`](https://nextjs.org/docs/pages/api-reference/functions/get-static-paths) function, Next.js by default tries to handle requests for non statically generated routes automatically, and creates a [Node.js runtime serverless function](https://nextjs.org/docs/app/building-your-application/rendering/edge-and-nodejs-runtimes) that is not compatible with Cloudflare Pages.

You can opt out of this behavior by specifying a [false `fallback`](https://nextjs.org/docs/pages/api-reference/functions/get-static-paths#fallback-false):

```diff
---
filename: pages/my-example-page/[slug].jsx
---
// ...

export async function getStaticPaths() {
    // ...

    return {
        paths,
+       fallback: false,
	}
}
```

{{<Aside type="warning">}}

Note that the `paths` array cannot be empty since an empty `paths` array causes Next.js to ignore the provided `fallback` value.

{{</Aside>}}