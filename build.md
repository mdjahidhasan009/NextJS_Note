# Build Process in Next.js

During the build time, Next.js performs several key operations to prepare your application for deployment. These operations optimize performance, generate static assets, and set up dynamic behavior. Here's an in-depth look:

## 1. File System Routing Setup
- **What happens**: Next.js analyzes your project’s `pages` or `app` directory to create a mapping of routes based on the file system structure.
- **Output**: A routing table that maps URLs to components.

## 2. Static Generation (SSG)
- **What happens**:
    - For pages using `getStaticProps` or `export const dynamic = 'force-static'` (in App Router):
        - Fetches all required data at build time.
        - Generates the static HTML for each page.
        - Saves these pages as `.html` files in the output directory.
- **Output**: Pre-rendered HTML and JSON files for each page.

## 3. Server-Side Rendering (SSR) Preparation
- **What happens**:
    - For pages using `getServerSideProps`, Next.js sets up placeholders and prepares dynamic rendering logic for runtime.
- **Output**: No pre-rendered HTML is created; instead, serverless functions or edge functions are prepared for deployment.

## 4. Incremental Static Regeneration (ISR) Setup
- **What happens**:
    - For pages using `revalidate` in `getStaticProps`:
        - Pre-renders the page like SSG.
        - Sets up revalidation logic for background regeneration of the page after the specified interval.
- **Output**: Static HTML initially, with regeneration configured for runtime.

## 5. Image Optimization Setup
- **What happens**:
    - Optimizes images referenced in the application using the `next/image` component.
    - Generates low-resolution placeholders (if enabled).
    - Prepares to dynamically optimize images at runtime.
- **Output**: Configuration files and metadata for image optimization.

## 6. Code Splitting and Bundling
- **What happens**:
    - Webpack (or the configured bundler) splits the code into smaller chunks to reduce the initial page load size:
        - Creates separate bundles for each page.
        - Extracts common dependencies into shared bundles.
- **Output**: Optimized JavaScript and CSS files, split into page-specific and shared chunks.

## 7. Static Asset Optimization
- **What happens**:
    - Optimizes CSS, JavaScript, and other assets like fonts.
    - Minifies JavaScript and CSS.
    - Adds hashes to file names for cache busting.
- **Output**: Optimized static files ready to be served by a CDN.

## 8. Middleware Compilation
- **What happens**:
    - Middleware logic (if defined in `middleware.ts` or `.js`) is compiled for runtime execution at the edge.
- **Output**: Middleware scripts optimized for edge execution.

## 9. Static Export (Optional)
- **What happens**:
    - If `next export` is used, Next.js generates a fully static site without any server-side functionality (e.g., no SSR or ISR).
- **Output**: Purely static HTML and assets.

## 10. Generating Metadata and Sitemaps
- **What happens**:
    - If metadata is defined in the `next/head` or via the App Router metadata API, Next.js generates static `<head>` information for each page.
- **Output**: Metadata files or embedded metadata in HTML.

## 11. Analyzing and Tree Shaking
- **What happens**:
    - Removes unused code (tree-shaking) and performs bundle analysis to further optimize the output.
- **Output**: Smaller and leaner JavaScript bundles.

## 12. Preparing for Deployment
- **What happens**:
    - Next.js packages the application, including:
        - Static files.
        - Serverless functions or edge functions.
        - Configurations for deployment platforms like Vercel.
- **Output**: A `.next` folder containing all assets, serverless functions, and metadata.



# References
* [Thinking in NextJS](https://www.stacklearner.com/my/workshops/thinking-in-nextjs?wid=18dd4467-6515-4d68-a7a1-6a5748c69054)
