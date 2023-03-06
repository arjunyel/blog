---
title: "tRPC with no latency or network errors"
description: "Using tRPC with Cloudflare Worker service bindings removes latency and network errors"
pubDate: "Mar 06 2023"
image: "/trpc-service-binding.png"
heroImage: "/trpc-service-binding.png"
tags: ["tRPC", "Cloudflare Workers", "Service Binding"]
---

Cloudflare Workers have a feature called Service bindings. Service Bindings allows you call another Cloudflare Worker without going over the network, eliminating latency and possible network errors.

An example use case is having our frontend in Cloudflare Pages call a backend tRPC worker. Separating our backend into a separate worker allows us to turn on Node compatibility without including the downsides in our frontend. It also lets us reduce the Pages function bundle size.

To enable this magic, we pass in our Service binding to the tRPC client's fetch option:

```typescript
createTRPCProxyClient<AppRouter>({
  links: [
    httpBatchLink({
      // Any URL will work since we are communicating with the worker directly
      url: "https://www.example.com/trpc",
      // Pass in the Service binding after binding it (lol)
      fetch: env.BACKEND.fetch.bind(env.BACKEND),
    }),
  ],
});
```
