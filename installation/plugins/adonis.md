---
description: '@adminjs/adonis'
---

# Adonis

`@adminjs/adonis` is an official integration with [AdonisJS](https://adonisjs.com/) and it's Lucid ORM. It supports AdonisJS v6+ and requires NodeJS v20.6.x+.

{% hint style="warning" %}
`@adminjs/adonis` currently only supports Lucid as it's ORM.  We will soon add support for other database adapters.
{% endhint %}

The easiest way to get started with AdonisJS integration is to follow their [official documentation](https://docs.adonisjs.com/guides/installation).&#x20;

Below you will find a simplified list of steps required to set up your AdminJS panel.

First of all, create your AdonisJS app if you haven't got one yet:

```bash
$ npm init adonisjs@latest -- -K=slim
```

Follow the steps from AdonisJS CLI to configure your application and once it's installed, navigate to the newly created directory.

```bash
$ cd <your_app_directory>
```

AdminJS requires you to install and configure `@adonisjs/session` and `@adonisjs/lucid`

```bash
$ npm install @adonisjs/session @adonisjs/lucid
$ node ace configure @adonisjs/session
$ node ace configure @adonisjs/lucid
```

This will create a configuration file for `@adonisjs/session` in  `config/session.ts`. The default configuration will allow you to sign into your admin panel in your development environment, but before you deploy to production, please see the [official documentation](https://docs.adonisjs.com/guides/session#session).

You should also configure Lucid in `config/database.ts`. Without this, you won't be able to start your app.

You will also need to install the core package of AdminJS:

```bash
$ npm install adminjs
```

Finally, install `@adminjs/adonis` which will add AdminJS integration into your application.

```bash
$ npm install @adminjs/adonis
$ node ace configure @adminjs/adonis
```

Three new files should appear in your codebase.

* **config/adminjs.ts** contains the configuration of Lucid adapter, Authentication and AdminJS
* **app/admin/component\_loader.ts** is a file where [ComponentLoader](../../ui-customization/writing-your-own-components.md) is instantiated and exported
* **app/admin/auth.ts** is a file where default authentication provider is created. Do note that by default, it will let everyone into your admin panel, so make sure you modify the `authenticate` method.

You should be able to start the application with `npm run dev` command and you should see the login page of AdminJS if you navigate to the default URL: [http://localhost:3333/admin](http://localhost:3333/admin/login)

### Authentication

Authentication can be configured in `config/adminjs.ts` under the `auth` option.

Default configuration:

```typescript
import authProvider from '../app/admin/auth.js'

// ...
auth: {
  enabled: true,
  provider: authProvider,
  middlewares: [],
}
```

The authentication can be disabled or enabled using the `enabled` flag.

[Provider](../../basics/authentication/) contains the logic for authenticating the users in your admin panel. The default provider uses `email` and `password` to authenticate users.

You may also configure any additional `middlewares` to routes that require authentication. Every custom middleware is run after the user's session is confirmed to exist.

### AdminJS Configuration

AdminJS can be configured in `config/adminjs.ts` under the `adminjs` option. It expects you to provide [AdminJSOptions](https://github.com/SoftwareBrothers/adminjs/blob/master/src/adminjs-options.interface.ts) object with the exception of `databases` which cannot be configured with `@adminjs/adonis` and you will have to configure every resource manually.

Resources configuration will be described in more detail in [Lucid ](adonis.md#lucid)section.

```typescript
adminjs: {
  rootPath: '/admin',
  loginPath: '/admin/login',
  logoutPath: '/admin/logout',
  componentLoader,
  resources: [],
  pages: {},
  locale: {
    availableLanguages: ['en'],
    language: 'en',
    translations: {
      en: {
        actions: {},
        messages: {},
        labels: {},
        buttons: {},
        properties: {},
        components: {},
        pages: {},
        ExampleResource: {
          actions: {},
          messages: {},
          labels: {},
          buttons: {},
          properties: {},
        },
      },
    },
  },
  branding: {
    companyName: 'AdminJS',
    theme: {},
  },
  settings: {
    defaultPerPage: 10,
  },
}
```

`adminjs` comes with some options already preconfigured. Note that `ExampleResource` in `translations` is just an example of how you would configure translations scoped to a specific resource. If your application has a Lucid `User` model that corresponds to `users` table, then `ExampleResource` could be renamed to `users`.

### Middlewares

Middlewares that will be applied to public routes can be configured in `config/adminjs.ts` under the `middlewares` option.

### Lucid

In order to add Lucid resources into your admin panel, you will first have to enable the adapter in `config/adminjs.ts`.

```typescript
adapter: {
  enabled: true
}
```

After you enable the adapter, you can start adding your Lucid models as resources to `adminjs#resources` in `config/adminjs.ts`:

```typescript
import { LucidResource } from '@adminjs/adonis'

import User from '../app/models/user.js'
import Profile from '../app/models/profile.js'

// ...
adminjs: {
  resources: [
    new LucidResource(User, 'postgres'),
    {
      resource: new LucidResource(Profile, 'postgres'),
      options: {},
    }
  ],
  // ...
}
```

**LucidResource** is an additional wrapper which uses Knex to fetch schema information about your database table. This is required to properly type and configure AdminJS resources. For every resource that you configure, it will query the database to get columns metadata. This is donly only once when you start your application.

**LucidResource** takes two arguments in it's constructor. The first is your Lucid model, and the second is your database connection name (this can be checked in `config/database.ts`).

There are two approaches to adding resources. You can either simply pass `new LucidResource(...)` and use the default configuration for that model, or you can pass a `{ resource: new LucidResource(...), options: {}` object with your custom [Resource configuration](https://github.com/SoftwareBrothers/adminjs/blob/master/src/backend/decorators/resource/resource-options.interface.ts#L48) in `options`.
