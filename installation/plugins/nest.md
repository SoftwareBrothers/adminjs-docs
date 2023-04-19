---
description: '@adminjs/nestjs'
cover: ../../.gitbook/assets/nestjs 1080p (1) (1).png
coverY: 0
---

# Nest

{% hint style="warning" %}
AdminJS is now ESM-only. If you plan to use NestJS in your project, NestJS doesn't work with ESM out of the box and you should be aware that you can run into issues. We suggest to use `@adminjs/express` instead. This guide will show you how you can set up the latest AdminJS version with NestJS but do note that this is just a workaround until NestJS maintainers decide to move to or at least support ESM.
{% endhint %}

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

{% hint style="warning" %}

{% endhint %}

Set `moduleResolution` to `nodenext` or `node16` in your `tsconfig.json`:

```json
{
  "compilerOptions": {
    "moduleResolution": "node16",
    "module": "commonjs",
    "target": "esnext",
    // ...
  }
}
```

Next, add Nest plugin code into `imports` of your `AppModule`:

{% code title="app.module.ts" %}
```typescript
import { Module } from '@nestjs/common'

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
    // AdminJS version 7 is ESM-only. In order to import it, you have to use dynamic imports.
    import('@adminjs/nestjs').then(({ AdminModule }) => AdminModule.createAdminAsync({
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
    })),
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

### Using AdminJS ESM packages

Since AdminJS version 7 and it's compatible packages are ESM-only, you cannot import them directly into your CommonJS Nest app. Instead, you have to use dynamic imports:

```typescript
const adminjsUploadFeature = await import('@adminjs/upload');
```
