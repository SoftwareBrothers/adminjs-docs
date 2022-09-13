---
description: '@adminjs/prisma'
cover: ../../.gitbook/assets/prisma.png
coverY: 0
---

# Prisma

{% hint style="info" %}
Before reading this article, make sure you have set up an AdminJS instance using one of the supported [Plugins](../plugins/).\
Additionally, you should have installed `@adminjs/prisma` as described in [Getting started](../getting-started.md) section.
{% endhint %}

This guide will assume you have set up Prisma using it's [documentation](https://www.prisma.io/docs/getting-started/quickstart) or [Nest.js documentation](https://docs.nestjs.com/recipes/prisma).

There are small differences in how you connect Prisma to Nest.js vs other plugins, so the guide will be split into two sections accordingly.

Example Prisma schema:

{% code title="prisma.schema" %}
```typescript
datasource db {
  provider = "mysql"
  url      = env("MYSQL_DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

model Publisher {
  id      Int      @id @default(autoincrement())
  email   String   @unique
  name    String?
}

```
{% endcode %}

### Standard

Make sure you have followed the tutorial for the framework you are using in the [Plugins](../plugins/) section.

The configuration for non-Nest.js plugins is basically the same for each one of them:

* You must instantiate a Prisma Client before creating `AdminJS` instance
* You must import `AdminJSPrisma` adapter and register it
* You must import the entities you want to use and pass them to AdminJS `resources` options

{% code title="app.ts" %}
```typescript
// ... other imports
import * as AdminJSPrisma from '@adminjs/prisma'
import { PrismaClient } from '@prisma/client'
import { DMMFClass } from '@prisma/client/runtime'
import { Category } from './category.entity'

const prisma = new PrismaClient()

AdminJS.registerAdapter({
  Resource: AdminJSPrisma.Resource,
  Database: AdminJSPrisma.Database,
})

// ... other code
const start = async () => {
  // `_baseDmmf` contains necessary Model metadata but it is a private method
  // so it isn't included in PrismaClient type
  const dmmf = ((prisma as any)._baseDmmf as DMMFClass)
  const adminOptions = {
    // We pass Publisher to `resources`
    resources: [{
      resource: { model: dmmf.modelMap.Publisher, client: prisma },
      options: {},
    }],
  }
  // Please note that some plugins don't need you to create AdminJS instance manually,
  // instead you would just pass `adminOptions` into the plugin directly,
  // an example would be "@adminjs/hapi"
  const admin = new AdminJS(adminOptions)
  // ... other code
}

start()
```
{% endcode %}

### Nest.js

Make sure you have set up your `app.module.ts` according to [Nest.js documentation](https://docs.nestjs.com/recipes/prisma) and you have followed [Nest.js plugin tutorial ](../plugins/nest.md)as well.

Your `app.module.ts` should have `imports` option which contains:

* `AdminModule.createAdminAsync({ ... }`

In your `app.module.ts` add these imports at the top of the file:

{% code title="app.module.ts" %}
```typescript
import * as AdminJSPrisma from '@adminjs/prisma'
import AdminJS from 'adminjs'
import { PrismaService } from './prisma.service' // PrismaService from Nest.js documentation
```
{% endcode %}

Following this, register `AdminJSPrisma` adapter somewhere after your imports:

{% code title="app.module.ts" %}
```typescript
AdminJS.registerAdapter({
  Resource: AdminJSPrisma.Resource,
  Database: AdminJSPrisma.Database,
})
```
{% endcode %}

This will allow you to pass Prisma models for AdminJS to load. If we use the `Publisher` entity that we used as en example earlier, you should import it into `app.module.ts` and pass it into `resources` in your `adminJsOptions`:

{% code title="app.module.ts" %}
```typescript
// ... other imports
import { Category } from './category.entity'
// ... other code
AdminModule.createAdminAsync({
  useFactory: () => {
    // Note: Feel free to contribute to this documentation if you find a Nest-way of
    // injecting PrismaService into AdminJS module
    const prisma = new PrismaService()
    // `_baseDmmf` contains necessary Model metadata but it is a private method
    // so it isn't included in PrismaClient type
    const dmmf = ((prisma as any)._baseDmmf as DMMFClass)
    return {
      adminJsOptions: {
        rootPath: '/admin',
        resources: [{
          resource: { model: dmmf.modelMap.Publisher, client: prisma },
          options: {},
        }],
      },
    }
  }
}),
// ... other code
```
{% endcode %}

