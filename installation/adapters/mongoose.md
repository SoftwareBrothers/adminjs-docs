---
description: '@adminjs/mongoose'
cover: ../../.gitbook/assets/mogoose.png
coverY: 0
---

# Mongoose

{% hint style="info" %}
Before reading this article, make sure you have set up an AdminJS instance using one of the supported [Plugins](../plugins/).\
Additionally, you should have installed `@adminjs/mongoose` as described in [Getting started](../getting-started.md) section.
{% endhint %}

This guide will assume you have set up Mongoose using it's [documentation](https://mongoosejs.com/docs/index.html) or [Nest.js documentation](https://docs.nestjs.com/recipes/sql-sequelize).

There are small differences in how you connect Mongoose to Nest.js vs other plugins, so the guide will be split into two sections accordingly.

Example model:

{% code title="category.entity.ts" %}
```typescript
import { model, Schema, Types } from 'mongoose'

export interface ICategory {
  title: string;
}

export const CategorySchema = new Schema<ICategory>(
  {
    title: { type: 'String', required: true },
  },
  { timestamps: true },
)

export const Category = model<ICategory>('Category', CategorySchema);
```
{% endcode %}

### Standard

Make sure you have followed the tutorial for the framework you are using in the [Plugins](../plugins/) section.

The configuration for non-Nest.js plugins is basically the same for each one of them:

* You must connect to Mongo before creating `AdminJS` instance
* You must import `AdminJSMongoose` adapter and register it
* You must import the entities you want to use and pass them to AdminJS `resources` options

{% code title="app.ts" %}
```typescript
// ... other imports
import mongoose from 'mongoose'
import * as AdminJSMongoose from '@adminjs/mongoose'
import { Category } from './category.entity'

AdminJS.registerAdapter({
  Resource: AdminJSMongoose.Resource,
  Database: AdminJSMongoose.Database,
})

// ... other code
const start = async () => {
  await mongoose.connect('<mongo db url>')
  const adminOptions = {
    // We pass Category to `resources`
    resources: [Category],
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

Make sure you have set up your `app.module.ts` according to [Nest.js documentation](https://docs.nestjs.com/techniques/mongodb) and you have followed [Nest.js plugin tutorial ](../plugins/nest.md)as well.

Your `app.module.ts` should have `imports` option which contains:

* `MongooseModule.forRoot('<mongo db url>')` to set up Mongoose
* `AdminModule.createAdminAsync({ ... }`

In your `app.module.ts` add these imports at the top of the file:

{% code title="app.module.ts" %}
```typescript
import * as AdminJSMongoose from '@adminjs/mongoose'
import AdminJS from 'adminjs'
```
{% endcode %}

Following this, register `AdminJSMongoose` adapter somewhere after your imports:

{% code title="app.module.ts" %}
```typescript
AdminJS.registerAdapter({
  Resource: AdminJSMongoose.Resource,
  Database: AdminJSMongoose.Database,
})
```
{% endcode %}

This will allow you to pass Mongoose models for AdminJS to load. If we use the `Category` entity that we used as en example earlier, you should import it into `app.module.ts` and pass it into `resources` in your `adminJsOptions`:

{% code title="app.module.ts" %}
```typescript
// ... other imports
import { Category } from './category.entity'
// ... other code
AdminModule.createAdminAsync({
  useFactory: () => ({
    adminJsOptions: {
      rootPath: '/admin',
      resources: [Category],
    },
  }),
}),
// ... other code
```
{% endcode %}

