Install `canary` version of `ppr` package:
```bash
npx create-next-app@canary my-app
```

At `next.config.ts` file, add the following code:
```ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  experimental: {
    ppr: true,
    dynamicIO: true, //Not for ppr, its for dynamic import optional 
  },
}
```












