Client side rendering cors 

* Long initial load time
* SEO(Search Engine Optimization) is not good
* Syncing state between server and client
* Need different strategies for caching

Server side rendering

* Server render the html and send it to the client along with the javascript
* Browser will take over the rendering
  * Will rehydrate the javascript for the client side rendering so it will be interactive
  * Will reconcile the state so that the client side rendering will be in sync with the server side rendering

But it is difficult to manually add server side rendering to an existing client side rendering app that's why Next.js 
came to the rescue. Next.js is a framework that allows you to add server side rendering to your react app with
minimal effort.

# Next.js
Next.js provide routing, layout, and other features that are not available in react. It also provide server side
rendering, static site generation, incremental static regeneration and partial pre rendering.

## SSG(Static Site Generation)
Best for blogs, marketing pages, documentation, terms and conditions, privacy policy, about us, contact us, etc site
where the content doesn't change often, like content added, deleted, or updated once a day or once a week or once a 
month.

If in a next.js page there is no api call, async call, or any dynamic data, search params, query params, dynamic routing, 
getServerSideProps, state, or any other dynamic data, then next.js will generate the html at build time, will store in
CDN(Content Delivery Network) and will serve the static html to the client.

* Generate the html at build time
* Serve the static html to the client
* Can be cached by a CDN(Content Delivery Network) for better performance

## ISR(Incremental Static Regeneration)
We can add revalidation date while fetching the data in getStaticProps. If the data is older than the revalidation date
then next.js will re-generate the html in the background after a certain period of time.

Combination of SSG and SSR. Ideal for pages where does not need fresh data on each request but need fresh data after a
certain period of time, like news, weather etc.

* Generate the html at build time
* Serve the static html to the client
* Re-generate the html in the background after a certain period of time
* Faster than SSR because the server has already generated the html

### APP Router
```ts
// app/posts/[id]/page.tsx
import { notFound } from 'next/navigation';

interface PostProps {
  params: { id: string };
}

export async function generateStaticParams() {
  // Fetch all possible IDs
  const res = await fetch('https://jsonplaceholder.typicode.com/posts');
  const posts = await res.json();

  return posts.map((post: { id: string }) => ({
    id: post.id.toString(),
  }));
}

export const revalidate = 60; // Revalidate every 60 seconds

export default async function PostPage({ params }: PostProps) {
  const res = await fetch(
    `https://jsonplaceholder.typicode.com/posts/${params.id}`,
    { next: { revalidate: 60 } }
  );

  if (!res.ok) {
    notFound();
  }

  const post = await res.json();

  return (
    <div>
      <h1>{post.title}</h1>
      <p>{post.body}</p>
    </div>
  );
}
```

### Page Router
```ts
// pages/posts/[id].js
import { useRouter } from 'next/router';

export async function getStaticPaths() {
  const res = await fetch('https://jsonplaceholder.typicode.com/posts');
  const posts = await res.json();

  const paths = posts.map((post) => ({
    params: { id: post.id.toString() },
  }));

  return { paths, fallback: 'blocking' };
}

export async function getStaticProps({ params }) {
  const res = await fetch(
    `https://jsonplaceholder.typicode.com/posts/${params.id}`
  );

  if (!res.ok) {
    return {
      notFound: true,
    };
  }

  const post = await res.json();

  return {
    props: { post },
    revalidate: 60, // Revalidate every 60 seconds
  };
}

export default function PostPage({ post }) {
  const router = useRouter();

  if (router.isFallback) {
    return <div>Loading...</div>;
  }

  return (
    <div>
      <h1>{post.title}</h1>
      <p>{post.body}</p>
    </div>
  );
}
```

#### Comparison

| Feature              | SSG                                               | ISR                              |
|----------------------|---------------------------------------------------|----------------------------------|
| ISR Implementation   | `revalidate` in `fetch` or `generateStaticParams` | `revalidate` in `getStaticProps` |
| Dynamic Routing      | File-based dynamic routes `[id]`                  | File-based dynamic routes `[id]` |
| Fallback Behavior    | `notFound()`	                                    | `fallback` in `getStaticPaths`   |

## SSR(Server Side Rendering)
Ideal for pages requiring fresh data on each request, like dashboard, products page etc.

* Generate the html on each request at the server
* Serve the html, along with the javascript, to the client
* SEO(Search Engine Optimization) is good
* Slow initial load time because the server has to generate the html on each request than SSG or ISR

## CSR(Client Side Rendering)

* Generate the html on the client
* Serve javascript to the client that will generate the html on the client

## PPR(Partial Pre Rendering)
In this stategy, some parts of the page are pre-rendered at build time and some parts are rendered at server side or 
at client side.

Ideal for pages where some parts of the page are static and some parts are dynamic, like header, footer, sidebar,
navigation, etc are static and the main content is dynamic. Or logged-in user information, cart information, etc are
dynamic and other parts of the page are static.

For e-commerce product description, price, image etc are rarely changed so they can be pre-rendered at build time and
stock availability, discount, etc are dynamic so they can be rendered at server side or at client side.

Need to change configuration in next.config.js as it is experimental feature
```ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
    experimental: {
        ppr: 'incremental',
    },
}

export default nextConfig
```
In page component
```ts
import { Suspense } from 'react'
import { StaticComponent, DynamicComponent, Fallback } from '@/app/ui'

export const experimental_ppr = true

export default function Page() {
    return (
        <>
            <StaticComponent />
        <Suspense fallback={<Fallback />}>
    <DynamicComponent />
    </Suspense>
    </>
)
}
```

| **Feature**               | **App Router (ISR with PPR)**                       | **Page Router (ISR with PPR)**                |
|---------------------------|-----------------------------------------------------|-----------------------------------------------|
| **Configuration**         | `export const revalidate = <seconds>`               | `revalidate` in `getStaticProps`              |
| **Dynamic Path Handling** | `generateStaticParams`                              | `getStaticPaths`                              |
| **Fallback Handling**     | `notFound()` for missing paths                      | `fallback` in `getStaticPaths`                |
| **ISR Fetching**          | `fetch(url, { next: { revalidate: <seconds> } })`   | Automatically handled in `getStaticProps`     |









# References
* [Thinking in NextJS](https://www.stacklearner.com/my/workshops/thinking-in-nextjs?wid=18dd4467-6515-4d68-a7a1-6a5748c69054)
