---
description: '@adminjs/fastify'
cover: ../../.gitbook/assets/fastify (1).png
coverY: 0
---

# Fastify

{% hint style="info" %}
Make sure you have installed AdminJS packages described in [Getting started](../getting-started.md) article.

```bash
$ yarn add adminjs @adminjs/fastify
```
{% endhint %}

To setup AdminJS panel with Fastify you need to have `fastify` installed and required peer dependencies:

```bash
$ yarn add fastify tslib
```

Afterwards, follow one of the examples below.

### Simple

{% tabs %}
{% tab title="Javascript" %}
{% code title="app.js" %}
```javascript
import AdminJSFastify from '@adminjs/fastify'
import AdminJS from 'adminjs'
import Fastify from 'fastify'

const PORT = 3000

const start = async () => {
  const app = Fastify()
  const admin = new AdminJS({
    databases: [],
    rootPath: '/admin'
  })

  await AdminJSFastify.buildRouter(
    admin,
    app,
  )
  
  app.listen({ port: PORT }, (err, addr) => {
    if (err) {
      console.error(err)
    } else {
      console.log(`AdminJS started on http://localhost:${PORT}${admin.options.rootPath}`)
    }
  })
}

start()
```
{% endcode %}
{% endtab %}

{% tab title="Typescript" %}
{% code title="app.ts" %}
```typescript
import AdminJSFastify from '@adminjs/fastify'
import AdminJS from 'adminjs'
import Fastify from 'fastify'

const PORT = 3000

const start = async () => {
  const app = Fastify()
  const admin = new AdminJS({
    databases: [],
    rootPath: '/admin'
  })

  await AdminJSFastify.buildRouter(
    admin,
    app,
  )
  
  app.listen({ port: PORT }, (err, addr) => {
    if (err) {
      console.error(err)
    } else {
      console.log(`AdminJS started on http://localhost:${PORT}${admin.options.rootPath}`)
    }
  })
}

start()
```
{% endcode %}
{% endtab %}
{% endtabs %}

### Authentication

To add authentication, you must use `AdminJSFastify.buildAuthenticatedRouter` instead of `AdminJSFastify.buildRouter`. Additionally, we must set up a session store to keep our session information. In the example below we will store our session in a Postgres table, we will also use `connect-pg-simple` to allow our session store to connect to the database.

{% tabs %}
{% tab title="Javascript" %}
Install additional dependencies:

```bash
$ yarn add @fastify/session connect-pg-simple
```

{% code title="app.js" %}
```javascript
import AdminJSFastify from '@adminjs/fastify'
import FastifySession from '@fastify/session'
import AdminJS from 'adminjs'
import Fastify from 'fastify'
import Connect from 'connect-pg-simple'

const ConnectSession = Connect(FastifySession)
const sessionStore = new ConnectSession({
  conObject: {
    connectionString: 'postgres://adminjs:adminjs@localhost:5435/adminjs',
    ssl: process.env.NODE_ENV === 'production',
  },
  tableName: 'session',
  createTableIfMissing: true,
})

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
  const app = Fastify()
  const admin = new AdminJS({
    databases: [],
    rootPath: '/admin'
  })
  
  // "secret" must be a string with at least 32 characters, example:
  const cookieSecret = 'sieL67H7GbkzJ4XCoH0IHcmO1hGBSiG5'
  await AdminJSFastify.buildAuthenticatedRouter(
    admin,
    {
      authenticate,
      cookiePassword: cookieSecret,
      cookieName: 'adminjs',
    },
    app,
    {
      store: sessionStore as any,
      saveUninitialized: true,
      secret: cookieSecret,
      cookie: {
        httpOnly: process.env.NODE_ENV === 'production',
        secure: process.env.NODE_ENV === 'production',
      },
    }
  )
  
  app.listen({ port: PORT }, (err, addr) => {
    if (err) {
      console.error(err)
    } else {
      console.log(`AdminJS started on http://localhost:${PORT}${admin.options.rootPath}`)

    }
  })
}

start()
```
{% endcode %}

As you may have noticed, the `authenticate` function compares credentials you submit in the form with a hardcoded `DEFAULT_ADMIN` object. In your case, you might want to modify `authenticate` function's logic to compare form credentials against real database objects.

{% hint style="info" %}
If you plan to copy-paste this example, make sure you set the database connection string to a functional one.
{% endhint %}
{% endtab %}

{% tab title="Typescript" %}
Install additional dependencies:

```bash
$ yarn add @fastify/session connect-pg-simple
$ yarn add -D @types/connect-pg-simple
```

{% code title="app.ts" %}
```typescript
import AdminJSFastify from '@adminjs/fastify'
import FastifySession from '@fastify/session'
import AdminJS from 'adminjs'
import Fastify from 'fastify'
import Connect from 'connect-pg-simple'

// Note: There are typing issues between Fastify Session and connect-pg-simple
// but the session will work correctly.
const ConnectSession = Connect(FastifySession as any)
const sessionStore = new ConnectSession({
  conObject: {
    connectionString: 'postgres://adminjs:adminjs@localhost:5435/adminjs',
    ssl: process.env.NODE_ENV === 'production',
  },
  tableName: 'session',
  createTableIfMissing: true,
})

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
  const app = Fastify()
  const admin = new AdminJS({
    databases: [],
    rootPath: '/admin'
  })
  
  // "secret" must be a string with at least 32 characters, example:
  const cookieSecret = 'sieL67H7GbkzJ4XCoH0IHcmO1hGBSiG5'
  await AdminJSFastify.buildAuthenticatedRouter(
    admin,
    {
      authenticate,
      cookiePassword: cookieSecret,
      cookieName: 'adminjs',
    },
    app,
    {
      store: sessionStore as any,
      saveUninitialized: true,
      secret: cookieSecret,
      cookie: {
        httpOnly: process.env.NODE_ENV === 'production',
        secure: process.env.NODE_ENV === 'production',
      },
    }
  )
  
  app.listen({ port: PORT }, (err, addr) => {
    if (err) {
      console.error(err)
    } else {
      console.log(`AdminJS started on http://localhost:${PORT}${admin.options.rootPath}`)

    }
  })
}

start()
```
{% endcode %}

As you may have noticed, the `authenticate` function compares credentials you submit in the form with a hardcoded `DEFAULT_ADMIN` object. In your case, you might want to modify `authenticate` function's logic to compare form credentials against real database objects.

{% hint style="info" %}
If you plan to copy-paste this example, make sure you set the database connection string to a functional one.
{% endhint %}
{% endtab %}
{% endtabs %}
