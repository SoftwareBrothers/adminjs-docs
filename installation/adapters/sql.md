---
description: '@adminjs/sql'
cover: ../../.gitbook/assets/sql.png
coverY: 0
---

# SQL

{% hint style="info" %}
Before reading this article, make sure you have set up an AdminJS instance using one of the supported [Plugins](../plugins/).\
Additionally, you should have installed `@adminjs/sql` as described in [Getting started](../getting-started.md) section.
{% endhint %}

`@adminjs/sql` is an official adapter for SQL databases which does not require you to use an ORM and allows you to simply connect your admin panel directly to the database.

### Currently supported databases

* PostgreSQL
* more coming soon

## Usage

Most of the setup is very similar to other AdminJS adapters. The main difference is that you must feed your database credentials to the Adapter instead of an ORM so that it can parse your database schema and run queries against your database.

First of all, import `Adapter`, `Resource` and `Database` from `@adminjs/sql` .

```typescript
import { Adapter, Resource, Database } from '@adminjs/sql'
import AdminJS from 'adminjs'
```

Afterwards, register the adapter and initialize it to let it read your database schema:

```typescript
// ...

AdminJS.registerAdapter({
  Database,
  Resource,
})

// ...

const db = await new Adapter('postgresql', {
  connectionString: '<your database url>',
  database: '<your database name>',
}).init()
```

When creating an `Adapter` instance, the first parameter is your database's dialect (currently only `postgresql` can be used) while the second parameter is an object with your database connection options. Please refer to [Knex documentation](https://knexjs.org/guide/#configuration-options) to find out how the configuration object should look like as `@adminjs/sql` is based on Knex.

Finally, create an `AdminJS` instance and either include `db` in `databases` option to load your entire database (not recommended) or define `resources` separately:

```typescript
const admin = new AdminJS({
  resources: [
    {
      resource: db.table('users'),
      options: {},
    },
  ],
  // databases: [db], <- not recommended
});
```

The rest of the configuration is identical to other plugins and adapters. If you are confused, you can use the example below as a reference:

```typescript
import AdminJS from 'adminjs'
import express from 'express'
import Plugin from '@adminjs/express'
import { Adapter, Database, Resource } from '@adminjs/sql'

AdminJS.registerAdapter({
  Database,
  Resource,
})

const start = async () => {
  const app = express()

  const db = await new Adapter('postgresql', {
    connectionString: 'postgres://adminjs:adminjs@localhost:5432/adminjs_panel',
    database: 'adminjs_panel',
  }).init();

  const admin = new AdminJS({
    resources: [
      {
        resource: db.table('users'),
        options: {},
      },
    ],
  });

  admin.watch()

  const router = Plugin.buildRouter(admin)

  app.use(admin.options.rootPath, router)

  app.listen(8080, () => {
    console.log('app started')
  })
}

start()
```

## Database Relations

Currently only `many-to-one` relation works out of the box if you specify foreign key constraints in your database. Other relations will require you to make UI/backend customizations. Please see our [documentation](https://docs.adminjs.co) to learn more.

## Enums

As of version `1.0.0` database enums aren't automatically detected and loaded. You can assign them manually in your resource options:

```typescript
// ...
  const admin = new AdminJS({
    resources: [{
      resource: db.table('users'),
      options: {
        properties: {
          role: {
            availableValues: [
              { label: 'Admin', value: 'ADMIN' },
              { label: 'Client', value: 'CLIENT' },
            ],
          },
        },
      },
    }],
  })
// ...
```

## Timestamps

If your database tables have automatically default-set timestamps (`created_at`, `updated_at`, etc) they will be visible in create/edit forms by default. You can hide them in resource options:

```typescript
// ...
  const admin = new AdminJS({
    resources: [{
      resource: db.table('users'),
      options: {
        properties: {
          created_at: { isVisible: false },
          updated_at: { isVisible: false },
        },
      },
    }],
  })
// ...
```
