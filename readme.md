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


## How vercel serve a Next.JS request?

<img src="./images/img.png" alt="How vercel serve a request?" />

Image Source: [Thinking in NextJS](https://www.stacklearner.com/my/workshops/thinking-in-nextjs?wid=18dd4467-6515-4d68-a7a1-6a5748c69054)

### Overview

When a page in a Next.js application uses **Server-Side Rendering (SSR)**, the HTML is generated on the server and 
stored in the CDN (Content Delivery Network). CDNs like Amazon S3, Cloudflare, or others cache the content. Metadata 
generated at build time helps the edge network/CDN decide whether to serve the request from the CDN cache or to forward
it to the server for fresh JavaScript generation.

### Request Handling

1. **Initial Request**:
  - A request is made to the nearest Vercel edge server/CDN server.
  - The edge network/server determines how to handle the request.

2. **Edge Network/CDN**:
  - Serves content from the nearest edge server to the client.

---

### Scenarios

#### 1. **Static Request**:
- If the data is static, the CDN serves the static HTML directly to the client.

#### 2. **Simple Tasks**:
- For tasks that can be handled by the edge server, the request is served directly from the **Edge Runtime**.
- The Edge Runtime is less powerful than Node.js runtime but suitable for lightweight operations like:
  - Sending emails
  - Making HTTPS requests
  - Queuing jobs

#### 3. **Complex Tasks / Dynamic HTML**:
- For complex tasks that cannot be handled by the edge server, the request is forwarded to a **Serverless Function**.
- The Serverless Function generates the required response.

---

### Serverless Functions

- **Definition**: Lightweight, on-demand functions that run on the server.
- **Runtime**: Supports Node.js runtime.
- **Lifecycle**:
  - Created dynamically upon request.
  - Destroyed after task completion.
- **Performance**:
  - Initialization takes 30ms to 500ms.
  - Suitable for short-running tasks.
  - Not recommended for long-running operations, as Vercel imposes restrictions on such use cases.
- **Use Cases**:
  - Handling dynamic content.
  - Performing server-side computations.


To make this things happen smoothly and optimize the application for maximum performance, NextJS needs engineering in
several places. Such as,

* Complex Build System => NextJS manages this
* Serverless Functions => Vercel manages this
* Edge Runtime => Vercel manages this
* Global CDN => Vercel manages this
* Caching => NextJS + Vercel work together to manage this
* File System Based Routing => NextJS manages this
* Application Configurations => We take those decision
* Boundaries => We take those decision

## The Three-Step Process in Next.js Deployment

### Step 1: Application

- **Design the Application**: Structure and design your application based on requirements.
- **Write Necessary Code**: Implement the features and functionalities using React and Next.js.
- **Setup Boundaries**: Define the application's constraints and what it will and won't do.
- **Add Necessary Configurations**: Configure Next.js settings such as routing, environment variables, and other project
  settings.

---

### Step 2: Build

- **Resolve Routes**: Next.js resolves and maps routes based on the `pages/` or `app/` directory.
- **Generate Static Pages**:
  - For **Static Site Generation (SSG)** and **Incremental Static Regeneration (ISR)**.
  - Static content is prepared at build time for better performance.
- **Code Splitting and Bundling**: Automatically splits the code and bundles it to optimize loading performance.
- **Middleware Compilation**: Middleware logic is compiled to handle specific requests dynamically at runtime.
- **Generate Metadata**: Generates metadata for efficient caching and routing across the infrastructure.

---

### Step 3: Infrastructure

- **Managed by Infrastructure**:
  - Most backend tasks like scaling, caching, and routing are handled automatically.
- **Vercel as Infrastructure Provider**:
  - Vercel hosts and manages the application with optimized performance.
- **Custom Infrastructure**:
  - You can also set up your own infrastructure if Vercel isn't being used.

---

### Additional Notes

- **Vercel's Role**:
  - Provides seamless integration with Next.js.
  - Optimized for Static Site Generation (SSG) and Incremental Static Regeneration (ISR).
  - Leverages CDNs for serving static content and serverless functions for dynamic tasks.
- **Infrastructure Flexibility**:
  - While Vercel is a popular choice, Next.js can also be deployed on other platforms like AWS, Google Cloud, or 
    self-hosted servers.


## Decision Making in Next.js Engineering

### Routing Patterns
1. **Nested Routing**:
  - Used for components like tabs, accordions, etc., where caching the component is beneficial.
2. **Parallel Routing**:
  - Commonly used in dashboards to display multiple independent sections simultaneously.
3. **Intercepting Routing**:
  - Useful when building components that are reused across multiple pages.

---

### Layout Design
Proper layout design is critical for improving page load time and overall user experience.
- **Efficient Loading**:
  - Avoid loading all components after fetching all necessary data.
  - Implement skeleton loading for components while data is being fetched.
  - Show components immediately as their data is resolved.
- **API Call Strategies**:
  - Use `Promise.all` to make parallel API calls for better performance.
  - Sequential API calls when dependencies exist between requests.
- **Importance**:
  - Layout design plays a crucial role in performance and concurrency.
  - Focus on:
    - Setting boundaries for components.
    - Leveraging **Client-Server Composition** for dynamic and static rendering.

---

### Rendering Strategies

1. **Static Content**:
  - Use **Static Site Generation (SSG)** for content that doesn’t change frequently.
2. **Dynamic Content**:
  - **Server-Side Rendering (SSR)**: For fresh data on every request.
  - **Incremental Static Regeneration (ISR)**: For content updated periodically.
  - **Partial Pre-Rendering (PPR)**: Combines static and server-side rendering for hybrid content.
  - **Client-Side Rendering (CSR)**: For client-only rendering.
---

### Caching Strategies

- **Where to Cache**:
  1. At the API route level.
  2. Use **HTTP Cache** for browser-level caching.
  3. Implement **LRU Cache** at the middleware level for efficient caching of frequently accessed data.
---

### Runtime Options
Pages can run on either **Edge** or **Node.js** runtimes. Choose the runtime based on the performance requirements and
rendering strategy.
- **Edge Runtime**:
  - For lightweight and latency-sensitive tasks.
- **Node.js Runtime**:
  - For more complex, server-side logic.

---

### Key Considerations
- Match the routing pattern and layout strategy to the application's requirements.
- Use appropriate rendering strategies based on the content's nature (static or dynamic).
- Leverage caching effectively at different levels to optimize performance.
- Choose the correct runtime (Edge or Node.js) for each page or API route.




# References
* [Thinking in NextJS](https://www.stacklearner.com/my/workshops/thinking-in-nextjs?wid=18dd4467-6515-4d68-a7a1-6a5748c69054)
