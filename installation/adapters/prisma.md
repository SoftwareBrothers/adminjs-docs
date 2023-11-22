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
import { Database, Resource, getModelByName } from '@adminjs/prisma'
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

AdminJS.registerAdapter({ Database, Resource })

// ... other code
const start = async () => {
  const adminOptions = {
    resources: [{
      resource: { model: getModelByName('Post'), client: prisma },
      options: {},
    }, {
      resource: { model: getModelByName('Profile'), client: prisma },
      options: {},
    }, {
      resource: { model: getModelByName('Publisher'), client: prisma },
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
import { Database, Resource, getModelByName } from '@adminjs/prisma'
import AdminJS from 'adminjs'

import { PrismaService } from './prisma.service.js' // PrismaService from Nest.js documentation
```
{% endcode %}

Following this, register `AdminJSPrisma` adapter somewhere after your imports:

{% code title="app.module.ts" %}
```typescript
AdminJS.registerAdapter({ Database, Resource })
```
{% endcode %}

This will allow you to pass Prisma models for AdminJS to load. If we use the `Publisher` entity that we used as en example earlier, you should import it into `app.module.ts` and pass it into `resources` in your `adminJsOptions`:

{% code title="app.module.ts" %}
```typescript
// ... other imports
import { Category } from './category.entity.js'
// ... other code
AdminModule.createAdminAsync({
  useFactory: () => {
    // Note: Feel free to contribute to this documentation if you find a Nest-way of
    // injecting PrismaService into AdminJS module
    const prisma = new PrismaService()

    return {
      adminJsOptions: {
        rootPath: '/admin',
        resources: [{
          resource: { model: getModelByName('Post'), client: prisma },
          options: {},
        }],
      },
    }
  }
}),
// ... other code
```
{% endcode %}

### Custom client module

In case your generated client is not under the default path, you can pass `clientModule` to each resource's configuration:

```typescript
// other imports
// your custom prisma module
import PrismaModule from '../prisma/client-prisma/index.js';

// ...

const prisma = new PrismaModule.PrismaClient();

// ...

// Notice `clientModule` per resource
const admin = new AdminJS({
  resources: [{
    resource: {
      model: getModelByName('Post', PrismaModule),
      client: prisma,
      clientModule: PrismaModule,
    },
  }],
});
```

\
