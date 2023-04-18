---
description: '@adminjs/koa'
cover: ../../.gitbook/assets/koa.png
coverY: 0
---

# Koa

{% hint style="info" %}
Make sure you have installed AdminJS packages described in [Getting started](../getting-started.md) article.

```bash
$ yarn add adminjs @adminjs/koa
```
{% endhint %}

To setup AdminJS panel with Koa you need to have `koa` installed and required peer dependencies:

```bash
$ yarn add koa @koa/router koa2-formidable
```

Afterwards, follow one of the examples below.

### Simple

{% tabs %}
{% tab title="Javascript" %}
{% code title="app.js" %}
```javascript
import AdminJS from 'adminjs'
import AdminJSKoa from '@adminjs/koa'
import Koa from 'koa'

const PORT = 3000

const start = async () => {
  const app = new Koa()
  const admin = new AdminJS({
    resources: [],
    rootPath: '/admin',
  })

  const router = AdminJSKoa.buildRouter(admin, app)

  app
    .use(router.routes())
    .use(router.allowedMethods())

   app.listen(PORT, () => {
     console.log(`AdminJS available at http://localhost:${PORT}${admin.options.rootPath}`)
   })
}

start()
```
{% endcode %}
{% endtab %}

{% tab title="Typescript" %}
Install additional dependencies:

```bash
$ yarn add -D @types/koa
```

{% code title="app.ts" %}
```typescript
import AdminJS from 'adminjs'
import AdminJSKoa from '@adminjs/koa'
import Koa from 'koa'

const PORT = 3000

const start = async () => {
  const app = new Koa()
  const admin = new AdminJS({
    resources: [],
    rootPath: '/admin',
  })

  const router = AdminJSKoa.buildRouter(admin, app)

  app
    .use(router.routes())
    .use(router.allowedMethods())

   app.listen(PORT, () => {
     console.log(`AdminJS available at http://localhost:${PORT}${admin.options.rootPath}`)
   })
}

start()
```
{% endcode %}
{% endtab %}
{% endtabs %}

### Authenticated

To add authentication, you must use `AdminJSKoa.buildAuthenticatedRouter` instead of `AdminJSKoa.buildRouter`.

{% tabs %}
{% tab title="Javascript" %}
{% code title="app.js" %}
```javascript
import AdminJS from 'adminjs'
import AdminJSKoa from '@adminjs/koa'
import Koa from 'koa'

const PORT = 3000

const DEFAULT_ADMIN = {
  email: 'admin@example.com',
  password: 'password',
}

const authenticate = async (email, password) => {
  if (email === DEFAULT_ADMIN.email && password === DEFAULT_ADMIN.password) {
    return Promise.resolve(DEFAULT_ADMIN)
  }
  return null
}

const start = async () => {
  const app = new Koa()

  const admin = new AdminJS({
    resources: [],
    rootPath: '/admin',
  })

  app.keys = ['your secret for koa cookie'];
  const router = AdminJSKoa.buildAuthenticatedRouter(
    admin,
    app,
    {
      authenticate,
      sessionOptions: {
        // You may configure your Koa session here
        httpOnly: process.env.NODE_ENV === 'production',
        secure: process.env.NODE_ENV === 'production',
        renew: true,
      },
    },
  )

  app
    .use(router.routes())
    .use(router.allowedMethods())

   app.listen(PORT, () => {
     console.log(`AdminJS available at http://localhost:${PORT}${admin.options.rootPath}`)
   })
}

start()
```
{% endcode %}

As you may have noticed, the `authenticate` function compares credentials you submit in the form with a hardcoded `DEFAULT_ADMIN` object. In your case, you might want to modify `authenticate` function's logic to compare form credentials against real database objects.
{% endtab %}

{% tab title="Typescript" %}
Install additional dependencies:

```bash
$ yarn add -D @types/koa
```

{% code title="app.ts" %}
```typescript
import AdminJS from 'adminjs'
import AdminJSKoa from '@adminjs/koa'
import Koa from 'koa'

const PORT = 3000

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

const start = async () => {
  const app = new Koa()

  const admin = new AdminJS({
    resources: [],
    rootPath: '/admin',
  })

  app.keys = ['your secret for koa cookie'];
  const router = AdminJSKoa.buildAuthenticatedRouter(
    admin,
    app,
    {
      authenticate,
      sessionOptions: {
        // You may configure your Koa session here
        httpOnly: process.env.NODE_ENV === 'production',
        secure: process.env.NODE_ENV === 'production',
        renew: true,
      },
    },
  )

  app
    .use(router.routes())
    .use(router.allowedMethods())

   app.listen(PORT, () => {
     console.log(`AdminJS available at http://localhost:${PORT}${admin.options.rootPath}`)
   })
}

start()
```
{% endcode %}

As you may have noticed, the `authenticate` function compares credentials you submit in the form with a hardcoded `DEFAULT_ADMIN` object. In your case, you might want to modify `authenticate` function's logic to compare form credentials against real database objects.
{% endtab %}
{% endtabs %}

