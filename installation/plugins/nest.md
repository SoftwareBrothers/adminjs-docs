---
description: '@adminjs/nestjs'
cover: ../../.gitbook/assets/nestjs 1080p (1) (1).png
coverY: 0
---

# Nest

{% hint style="info" %}
Make sure you have installed AdminJS packages described in [Getting started](../getting-started.md) article.

```bash
$ yarn add adminjs @adminjs/nestjs
```
{% endhint %}

As of version `5.x.x` of `@adminjs/nestjs`, the plugin only supports Nest servers that use Express. Fastify Nest servers are not currently supported.

{% hint style="info" %}
If you are starting a new project, install `nest-cli` and bootstrap the project:

```bash
$ npm i -g @nestjs/cli
$ nest new
```
{% endhint %}

`@adminjs/nestjs` uses `@adminjs/express` to set up AdminJS, because of this you have to additionally install it's dependencies:

```bash
$ yarn add @adminjs/express express-session express-formidable
```

Next, add Nest plugin code into `imports` of your `AppModule`:

{% code title="app.module.ts" %}
```typescript
import { Module } from '@nestjs/common'
import { AdminModule } from '@adminjs/nestjs'

import { AppController } from './app.controller.js'
import { AppService } from './app.service.js'

const DEFAULT_ADMIN = {
  email: 'admin@example.com',
  password: 'password',
}

const authenticate = async (email: string, password: string) => {
  if (email === DEFAULT_ADMIN.email && password === DEFAULT_ADMIN.password) {
    return Promise.resolve(DEFAULT_ADMIN)
  }
  return null
}

@Module({
  imports: [
    AdminModule.createAdminAsync({
      useFactory: () => ({
        adminJsOptions: {
          rootPath: '/admin',
          resources: [],
        },
        auth: {
          authenticate,
          cookieName: 'adminjs',
          cookiePassword: 'secret'
        },
        sessionOptions: {
          resave: true,
          saveUninitialized: true,
          secret: 'secret'
        },
      }),
    }),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```
{% endcode %}

Now you should be able to start your AdminJS application:

```bash
$ nest start
```

AdminJS panel will be available under `http://localhost:3000/admin` if you have followed this example thoroughly.

### How to remove authentication?

If you don't want your admin panel to require authentication, simply follow the example above, but remove `auth` and `sessionOptions` configuration, for example:

```typescript
    AdminModule.createAdminAsync({
      useFactory: () => ({
        adminJsOptions: {
          rootPath: '/admin',
          resources: [],
        },
      }),
    }),
```
