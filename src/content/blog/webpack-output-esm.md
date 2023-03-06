---
title: "Output ES Modules with Webpack"
description: "Here's the Webpack config to output ES Modules (ESM)"
pubDate: "Mar 06 2023"
tags: ["Webpack", "es modules", "esm"]
---

I was having trouble finding exactly how to output ES Modules with Webpack 5. Here's the Webpack configuration I used:

```typescript
{
  ...config,
  experiments: {
   futureDefaults: true,
   outputModule: true,
  },
  output: {
   ...config.output,
   module: true,
   chunkFormat: 'module',
  },
 }
```
