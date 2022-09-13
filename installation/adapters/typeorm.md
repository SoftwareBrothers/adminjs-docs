---
description: '@adminjs/typeorm'
cover: ../../.gitbook/assets/typeorm.png
coverY: 0
---

# TypeORM

{% hint style="info" %}
Before reading this article, make sure you have set up an AdminJS instance using one of the supported [Plugins](../plugins/).\
Additionally, you should have installed `@adminjs/typeorm` as described in [Getting started](../getting-started.md) section.
{% endhint %}

This guide will assume you have set up TypeORM using it's [documentation](https://typeorm.io/#quick-start) or [Nest.js documentation](https://docs.nestjs.com/recipes/sql-typeorm).

`@adminjs/typeorm` currently only supports entities that extend `BaseEntity` as shown on the example below:

{% code title="organization.entity.ts" %}
```typescript
import { BaseEntity, Column, Entity, PrimaryGeneratedColumn } from 'typeorm'

@Entity({ name: 'organizations' })
export class Organization extends BaseEntity {
  @PrimaryGeneratedColumn()
  public id: number;

  @Column()
  public name: string;
}
```
{% endcode %}

There are small differences in how you connect TypeORM to Nest.js vs other plugins, so the guide will be split into two sections accordingly.

### Standard

Make sure you have followed the tutorial for the framework you are using in the [Plugins](../plugins/) section.

The configuration for non-Nest.js plugins is basically the same for each one of them:

* You must import the data source and initialize it
* You must import `AdminJSTypeorm` adapter and register it
* You must import the entities you want to use and pass them to AdminJS `resources` options

{% code title="app.ts" %}
```typescript
// ... other imports
import * as AdminJSTypeorm from '@adminjs/typeorm'
import dataSource from './path/to/your/datasource'
import { Organization } from './organization.entity'

AdminJS.registerAdapter({
  Resource: AdminJSTypeorm.Resource,
  Database: AdminJSTypeorm.Database,
})

// ... other code
const start = async () => {
  // Make sure you initialize the data source before you create your AdminJS instance
  await dataSource.initialize()
  const adminOptions = {
    // We pass Organization to `resources`
    resources: [Organization],
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

Make sure you have set up your `app.module.ts` according to [Nest.js documentation](https://docs.nestjs.com/recipes/sql-typeorm) and you have followed [Nest.js plugin tutorial ](../plugins/nest.md)as well.

Your `app.module.ts` should have `imports` option which contains:

* `TypeOrmModule.forRoot(params)` to set up TypeORM
* `AdminModule.createAdminAsync({ ... }`

In your `app.module.ts` add these imports at the top of the file:

{% code title="app.module.ts" %}
```typescript
import * as AdminJSTypeorm from '@adminjs/typeorm'
import AdminJS from 'adminjs'
```
{% endcode %}

Following this, register `AdminJSTypeorm` adapter somewhere after your imports:

{% code title="app.module.ts" %}
```typescript
AdminJS.registerAdapter({
  Resource: AdminJSTypeorm.Resource,
  Database: AdminJSTypeorm.Database,
})
```
{% endcode %}

This will allow you to pass TypeORM models for AdminJS to load. If we use the `Organization` entity that we used as en example earlier, you should import it into `app.module.ts` and pass it into `resources` in your `adminJsOptions`:

{% code title="app.module.ts" %}
```typescript
// ... other imports
import { Organization } from './organization.entity'
// ... other code
AdminModule.createAdminAsync({
  useFactory: () => ({
    adminJsOptions: {
      rootPath: '/admin',
      resources: [Organization],
    },
  }),
}),
// ... other code
```
{% endcode %}

