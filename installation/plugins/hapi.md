---
description: '@adminjs/hapi'
cover: ../../.gitbook/assets/hapi (1).png
coverY: 0
---

# Hapi

{% hint style="info" %}
Make sure you have installed AdminJS packages described in [Getting started](../getting-started.md) article.

```bash
$ yarn add adminjs @adminjs/hapi
```
{% endhint %}

To setup AdminJS panel with Hapi you need to have `@hapi/hapi` installed and required peer dependencies:

```bash
$ yarn add @hapi/hapi @hapi/boom @hapi/cookie @hapi/inert
```

Afterwards, follow one of the examples below.

### Simple

{% tabs %}
{% tab title="Javascript" %}
{% code title="app.js" %}
```javascript
const { default: AdminJSHapi } = require('@adminjs/hapi')
const Hapi = require('@hapi/hapi')

const PORT = 3000

const start = async () => {
  const server = Hapi.server({ port: PORT })

  const adminOptions = {
    resources: [],
    rootPath: '/admin',
    auth: {
      isSecure: process.env.NODE_ENV === 'production',
    },
    registerInert: true,
  }

  await server.register({
    plugin: AdminJSHapi,
    options: adminOptions,
  })

  await server.start();
  console.log(`AdminJS available at ${server.info.uri}${adminOptions.rootPath}`);
}

start()
```
{% endcode %}

Now you can start your AdminJS application:

```bash
$ node app.js
```
{% endtab %}

{% tab title="Typescript" %}
Install `ts-node`:

```bash
$ yarn add -D ts-node
```

{% code title="app.ts" %}
```typescript
import AdminJSHapi, { ExtendedAdminJSOptions } from '@adminjs/hapi'
import Hapi from '@hapi/hapi'

const PORT = 3000

const start = async () => {
  const server = Hapi.server({ port: PORT })

  const adminOptions: ExtendedAdminJSOptions = {
    resources: [],
    rootPath: '/admin',
    auth: {
      isSecure: process.env.NODE_ENV === 'production',
    },
    registerInert: true,
  }

  await server.register<ExtendedAdminJSOptions>({
    plugin: AdminJSHapi,
    options: adminOptions,
  })

  await server.start();
  console.log(`AdminJS available at ${server.info.uri}${adminOptions.rootPath}`);
}

start()
```
{% endcode %}

Now you can start your AdminJS application:

```bash
$ ts-node app.ts
```
{% endtab %}
{% endtabs %}

### Authentication

To add authentication, all you have to do is extend `auth` config with `authenticate`, `cookieName` and `cookiePassword`.

{% tabs %}
{% tab title="Javascript" %}
{% code title="app.js" %}
```javascript
const { default: AdminJSHapi } = require('@adminjs/hapi')
const Hapi = require('@hapi/hapi')

const PORT = 3000

const DEFAULT_ADMIN = {
  email: 'admin@example.com',
  password: 'password',
  test: 'abc',
}

const authenticate = async (email, password) => {
  if (email === DEFAULT_ADMIN.email && password === DEFAULT_ADMIN.password) {
    return Promise.resolve(DEFAULT_ADMIN)
  }
  return null
}

const start = async () => {
  const server = Hapi.server({ port: PORT })

  // "secret" must be a string with at least 32 characters, example:
  const cookieSecret = 'sieL67H7GbkzJ4XCoH0IHcmO1hGBSiG5'
  const adminOptions = {
    resources: [],
    rootPath: '/admin',
    auth: {
      isSecure: process.env.NODE_ENV === 'production',
      authenticate,
      cookieName: 'adminjs',
      cookiePassword: cookieSecret,
    },
    registerInert: true,
  }

  await server.register({
    plugin: AdminJSHapi,
    options: adminOptions,
  })

  await server.start();
  console.log(`AdminJS available at ${server.info.uri}${adminOptions.rootPath}`);
}

start()
```
{% endcode %}

Now you can start your AdminJS application:

```bash
$ node app.js
```
{% endtab %}

{% tab title="Second Tab" %}
Install `ts-node`:

```bash
$ yarn add -D ts-node
```

{% code title="app.ts" %}
```typescript
import AdminJSHapi, { ExtendedAdminJSOptions } from '@adminjs/hapi'
import Hapi from '@hapi/hapi'

const PORT = 3000

const DEFAULT_ADMIN = {
  email: 'admin@example.com',
  password: 'password',
}

const authenticate = async (email: string, password: string): Promise<Record<string, unknown> | null> => {
  if (email === DEFAULT_ADMIN.email && password === DEFAULT_ADMIN.password) {
    return Promise.resolve(DEFAULT_ADMIN)
  }
  return null
}

const start = async () => {
  const server = Hapi.server({ port: PORT })

  // "secret" must be a string with at least 32 characters, example:
  const cookieSecret = 'sieL67H7GbkzJ4XCoH0IHcmO1hGBSiG5'
  const adminOptions: ExtendedAdminJSOptions = {
    resources: [],
    rootPath: '/admin',
    auth: {
      isSecure: process.env.NODE_ENV === 'production',
      authenticate,
      cookieName: 'adminjs',
      cookiePassword: cookieSecret,
    },
    registerInert: true,
  }

  await server.register<ExtendedAdminJSOptions>({
    plugin: AdminJSHapi,
    options: adminOptions,
  })

  await server.start();
  console.log(`AdminJS available at ${server.info.uri}${adminOptions.rootPath}`);
}

start()
```
{% endcode %}

Now you can start your AdminJS application:

```bash
$ ts-node app.ts
```
{% endtab %}
{% endtabs %}
