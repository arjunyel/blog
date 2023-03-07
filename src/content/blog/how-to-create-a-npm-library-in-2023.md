---
title: "How to create a npm library in 2023"
description: "Deno's dnt tool lets you to create an npm library that works in every environment" pubDate: "Mar 07 2023"
image: "/creating-a-npm-library-in-2023.png"
heroImage: "/creating-a-npm-library-in-2023.png"
tags: ["npm library", "deno", "node", "dnt"]
---

Creating an npm library in 2023 can feel daunting.

- It has to work in every JavaScript environment, such as Deno, Node, and Cloudflare Workers.
- There needs to be TypeScript typings.
- It has to be bundled in both CommonJS and ES Modules format.

Thankfully there's a wonderful project by the [Deno](https://deno.land/) team called [dnt](https://github.com/denoland/dnt). Dnt lets you write your library with Deno, run your tests in Node and Deno, then compiles it into an npm package. Deno has a top-notch developer experience with native TypeScript support and a [standard library based on Go's](https://deno.land/manual/basics/standard_library).

Dnt handles all the bundling for you and lets you polyfill things for different platforms. For example, I wrote the [JavaScript SDK for Lago](https://github.com/getlago/lago-javascript-client), an open-source billing platform, using dnt. Deno supports [`URLPattern`](https://developer.mozilla.org/en-US/docs/Web/API/URLPattern) but Node does not. With dnt it's as simple as telling the build script for all instances of `URLPattern` you see, replace it with the `urlpattern-polyfill` npm package.

Here is the full script:

```typescript
import { build, emptyDir } from "https://deno.land/x/dnt@0.32.0/mod.ts";

await emptyDir("./npm");

await build({
  entryPoints: ["./mod.ts"],
  outDir: "./npm",
  typeCheck: false,
  shims: {
    // see JS docs for overview and more options
    deno: "dev",
    custom: [
      {
        package: {
          name: "urlpattern-polyfill",
          version: "^6.0.2",
        },
        globalNames: [
          {
            name: "URLPattern",
            exportName: "URLPattern",
          },
        ],
      },
    ],
  },
  compilerOptions: {
    target: "Latest",
    lib: ["dom", "es2022"],
  },
  package: {
    // package.json properties
    name: "lago-javascript-client",
    sideEffects: false,
    version: "v0.23.0-beta",
    description: "Lago JavaScript API Client",
    repository: {
      type: "git",
      url: "git+https://github.com/getlago/lago-javascript-client.git",
    },
    keywords: ["Lago", "Node", "API", "Client"],
    contributors: ["Lovro Colic", "Jérémy Denquin", "Arjun Yelamanchili"],
    license: "MIT",
    bugs: {
      url: "https://github.com/getlago/lago-javascript-client/issues",
    },
    homepage: "https://github.com/getlago/lago-javascript-client#readme",
  },
});

// post build steps
Deno.copyFileSync("LICENSE", "npm/LICENSE");
Deno.copyFileSync("README.md", "npm/README.md");
```

You can see the [code output on npm](https://www.npmjs.com/package/lago-javascript-client?activeTab=explore).
