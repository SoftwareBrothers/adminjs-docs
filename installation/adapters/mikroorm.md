---
description: '@adminjs/mikroorm'
cover: ../../.gitbook/assets/mikroorm.png
coverY: 0
---

# MikroORM

{% hint style="info" %}
Before reading this article, make sure you have set up an AdminJS instance using one of the supported [Plugins](../plugins/).\
Additionally, you should have installed `@adminjs/mikroorm` as described in [Getting started](../getting-started.md) section.
{% endhint %}

This guide will assume you have set up MikroORM using it's [documentation](https://mikro-orm.io/docs/installation) or [Nest.js documentation](https://docs.nestjs.com/recipes/mikroorm).

There are small differences in how you connect MikroORM to Nest.js vs other plugins, so the guide will be split into two sections accordingly.

Example model:

{% code title="owner.entity.ts" %}
```typescript
import { v4 } from 'uuid'
import { BaseEntity, Entity, PrimaryKey, Property } from '@mikro-orm/core'

export interface IOwner {
  firstName: string;
  lastName: string;
  age: number;
}

@Entity({ tableName: 'owners' })
export class Owner extends BaseEntity<Owner, 'id'> implements IOwner {
  @PrimaryKey({ columnType: 'uuid' })
  id = v4()

  @Property({ fieldName: 'first_name', columnType: 'text' })
  firstName: string

  @Property({ fieldName: 'last_name', columnType: 'text' })
  lastName: string

  @Property({ fieldName: 'age', columnType: 'integer' })
  age: number

  @Property({ fieldName: 'created_at', columnType: 'timestamptz' })
  createdAt: Date = new Date()

  @Property({
    fieldName: 'updated_at',
    columnType: 'timestamptz',
    onUpdate: () => new Date(),
  })
  updatedAt: Date = new Date()
}
```
{% endcode %}

### Standard

Make sure you have followed the tutorial for the framework you are using in the [Plugins](../plugins/) section.

The configuration for non-Nest.js plugins is basically the same for each one of them:

* You must initialize MikroORM before creating `AdminJS` instance
* You must import `AdminJSMikroORM` adapter and register it
* You must import the entities you want to use and pass them to AdminJS `resources` options

{% code title="app.ts" %}
```typescript
// ... other imports
import { MikroORM } from '@mikro-orm/core'
import * as AdminJSMikroORM from '@adminjs/mikroorm'
import { Owner } from './owner.entity.js'

AdminJS.registerAdapter({
  Resource: AdminJSMikroORM.Resource,
  Database: AdminJSMikroORM.Database,
})

  // Note: `config` is your MikroORM configuration as described in it's docs
const config = {
  entities: [Owner],
  dbName: 'adminjs',
  type: 'postgresql',
  clientUrl: 'postgres://adminjs:adminjs@localhost:5435/adminjs',
}

// ... other code
const start = async () => {
  const orm = await MikroORM.init(config)
  const adminOptions = {
    // We pass Owner to `resources`
    resources: [{
      resource: { model: Owner, orm },
      options: {}
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

Make sure you have set up your `app.module.ts` according to [Nest.js documentation](https://docs.nestjs.com/recipes/mikroorm) and you have followed [Nest.js plugin tutorial ](../plugins/nest.md)as well.

Your `app.module.ts` should have `imports` option which contains:

* `MikroOrmModule.forRoot(...)` to set up MikroORM:

```typescript
// Note: this is a default configuration from Nest.js documentation
MikroOrmModule.forRoot({
  entities: ['./dist/entities'],
  entitiesTs: ['./src/entities'],
  dbName: 'my-db-name.sqlite3',
  type: 'sqlite',
})
```

* `AdminModule.createAdminAsync({ ... }`

In your `app.module.ts` add these imports at the top of the file:

{% code title="app.module.ts" %}
```typescript
import * as AdminJSMikroORM from '@adminjs/mikroorm'
import AdminJS from 'adminjs'
```
{% endcode %}

Following this, register `AdminJSMikroORM` adapter somewhere after your imports:

{% code title="app.module.ts" %}
```typescript
AdminJS.registerAdapter({
  Resource: AdminJSMikroORM.Resource,
  Database: AdminJSMikroORM.Database,
})
```
{% endcode %}

This will allow you to pass MikroORM models for AdminJS to load. If we use the `Owner` entity that we used as en example earlier, you should import it into `app.module.ts` and pass it into `resources` in your `adminJsOptions`:

{% code title="app.module.ts" %}
```typescript
// ... other imports
import { Owner } from './owner.entity.js'
// ... other code
AdminModule.createAdminAsync({
  useFactory: () => ({
    adminJsOptions: {
      rootPath: '/admin',
      resources: [{
        resource: { model: Owner, orm },
        options: {}
      }],
    },
  }),
}),
// ... other code
```
{% endcode %}

