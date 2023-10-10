---
title: "How I work with unfamiliar libraries"
description: "When working with unfamiliar libraries, I find public examples using Sourcegraph"
pubDate: "Mar 06 2023"
image: "../../assets/working-with-unfamiliar-libraries.png"
heroImage: "../../assets/working-with-unfamiliar-libraries.png"
tags: ["sourcegraph"]
---

When working with libraries I haven't worked with before, examples help me the most. Using [Sourcegraph](https://sourcegraph.com/), I can search millions of public repositories for where they are importing or using the same library as me. Usually, you'll find someone doing what you want to do!

For example, I tried to use [Astro's `<Picture />` component](https://docs.astro.build/en/guides/integrations-guide/image/) for my blog but didn't know how to use the `sizes` attribute. Using Sourcegraph, I [found a test](https://github.com/withastro/astro/blob/af05a4fa4637bad9b20b9928e6d2c2f894aa9139/packages/integrations/image/test/fixtures/basic-picture/src/pages/index.astro) doing exactly what I needed.

```typescript
<Picture
  src={heroImage}
  width={720}
  height={360}
  alt=""
  sizes="100vw"
  widths={[720]}
  aspectRatio={720 / 360}
  formats={["avif", "webp"]}
  loading={"eager"}
/>
```
