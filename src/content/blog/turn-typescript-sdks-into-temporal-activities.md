---
title: "Turn TypeScript SDKs Into Temporal Activities"
description: "Easily turn SDKs such as Stripe or Lago into Temporal Activities"
pubDate: "Mar 18 2023"
updatedDate: "Apr 5 2023"
image: "/turn-typescript-sdks-into-temporal-activities.png"
heroImage: "/turn-typescript-sdks-into-temporal-activities.png"
tags: ["temporal", "typescript", "sdk"]
---

[Temporal Activities](https://docs.temporal.io/activities) must be functions, so we can't easily pass around SDKs like we are used to. I was manually creating function wrappers over each SDK method until [Roey Berman](https://github.com/bergundy/) from the Temporal team helped me with this function that turned SDKs into Activities.

Here's an example using the [Lago SDK](https://github.com/getlago/lago-javascript-client) and the [Stripe SDK](https://github.com/stripe/stripe-node), starting from the Temporal TypeScript [dependency injection example](https://github.com/temporalio/samples-typescript/tree/main/activities-dependency-injection):

```typescript
import type { Api } from "lago-javascript-client";
import type Stripe from "stripe";

// Add prefix to object key https://stackoverflow.com/a/70387184/4717424
type AddPrefixToObject<T, P extends string> = {
  [K in keyof T as K extends string ? `${P}${K}` : never]: T[K];
};

// Get all object keys where values are type V https://stackoverflow.com/a/56874389/4717424
type KeysMatching<T extends object, V> = {
  [K in keyof T]-?: T[K] extends V ? K : never;
}[keyof T];

// SDK after processing
type ProcessedSdk<T extends object, Prefix extends string> = AddPrefixToObject<
  Pick<T, KeysMatching<T, Function>>,
  Prefix
>;

/**
 * Get all functions from SDK, add prefix to key, bind them, and return new object
 */
function turnSdkToObject<T extends object, Prefix extends string>(
  sdkObj: T,
  prefix: Prefix,
  type: "object" | "class"
) {
  if (type === "object") {
    return Object.fromEntries(
      Object.entries(sdkObj)
        .filter(([, v]) => typeof v === "function")
        .map(([name, method]: [string, Function]) => [
          `${prefix}${name}`,
          method.bind(sdkObj),
        ])
    ) as ProcessedSdk<T, Prefix>;
  }
  // If SDK is class https://stackoverflow.com/a/52592900/4717424
  return Object.getOwnPropertyNames(Object.getPrototypeOf(sdkObj))
    .filter(
      (name) =>
        name !== "constructor" && typeof (sdkObj as any)[name] === "function"
    )
    .reduce((methodCollection, methodName) => {
      (methodCollection as any)[`${prefix}${methodName}`] = (sdkObj as any)[
        methodName
      ].bind(sdkObj);
      return methodCollection;
    }, {} as ProcessedSdk<T, Prefix>);
}

export const createActivities = ({
  lagoSDK,
  stripeSDK,
}: {
  lagoSDK: Api<unknown>;
  stripeSDK: Stripe;
}) => ({
  ...turnSdkToObject(stripeSDK.customers, "stripe_customers_", "class"),
  ...turnSdkToObject(lagoClient.customers, "lago_customers_", "object"),
});
```

We add a prefix in case SDKs have overlapping method names. Now we can use SDKs in our Temporal Activities with minimal manual work.

```typescript
const { stripe_customers_create, lago_customers_createCustomer } =
  proxyActivities<ReturnType<typeof createActivities>>({
    startToCloseTimeout: "1 minute",
  });
```
