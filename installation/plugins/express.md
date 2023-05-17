---
description: '@adminjs/express'
cover: ../../.gitbook/assets/expressjs.png
coverY: 0
---

# Express

{% hint style="info" %}
Make sure you have installed AdminJS packages described in [Getting started](../getting-started.md) article.

```bash
$ yarn add adminjs @adminjs/express
```
{% endhint %}

To setup AdminJS panel with Express.js you need to have `express` installed and required peer dependencies:

```bash
$ yarn add express tslib express-formidable express-session
```

Afterwards, follow one of the examples below.

### Simple

{% tabs %}
{% tab title="Javascript" %}
{% code title="app.js" %}
```javascript
import AdminJS from 'adminjs'
import AdminJSExpress from '@adminjs/express'
import express from 'express'

const PORT = 3000

const start = async () => {
  const app = express()

  const admin = new AdminJS({})

  const adminRouter = AdminJSExpress.buildRouter(admin)
  app.use(admin.options.rootPath, adminRouter)

  app.listen(PORT, () => {
    console.log(`AdminJS started on http://localhost:${PORT}${admin.options.rootPath}`)
  })
}

start()
```
{% endcode %}
{% endtab %}

{% tab title="Typescript" %}
Install Express types:

```bash
$ yarn add -D @types/express
```

{% code title="app.ts" %}
```typescript
import AdminJS from 'adminjs'
import AdminJSExpress from '@adminjs/express'
import express from 'express'

const PORT = 3000

const start = async () => {
  const app = express()

  const admin = new AdminJS({})

  const adminRouter = AdminJSExpress.buildRouter(admin)
  app.use(admin.options.rootPath, adminRouter)

  app.listen(PORT, () => {
    console.log(`AdminJS started on http://localhost:${PORT}${admin.options.rootPath}`)
  })
}

start()
```
{% endcode %}

\
Now you can start your AdminJS application.
{% endtab %}
{% endtabs %}

### Authenticated

To add authentication, you must use `AdminJSExpress.buildAuthenticatedRouter` instead of `AdminJSExpress.buildRouter`. Additionally, we must set up a session store to keep our session information. In the example below we will store our session in a Postgres table, we will also use `connect-pg-simple` to allow our session store to connect to the database.

{% tabs %}
{% tab title="Javascript" %}


Install additional dependencies:

```bash
$ yarn add connect-pg-simple
```

{% code title="app.js" %}
```javascript
import AdminJS from 'adminjs'
import * as AdminJSExpress from '@adminjs/express'
import express from 'express'
import Connect from 'connect-pg-simple'
import session from 'express-session'

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
  const app = express()

  const admin = new AdminJS({})

  const ConnectSession = Connect(session)
  const sessionStore = new ConnectSession({
    conObject: {
      connectionString: 'postgres://adminjs:@localhost:5432/adminjs',
      ssl: process.env.NODE_ENV === 'production',
    },
    tableName: 'session',
    createTableIfMissing: true,
  })

  const adminRouter = AdminJSExpress.buildAuthenticatedRouter(
    admin,
    {
      authenticate,
      cookieName: 'adminjs',
      cookiePassword: 'sessionsecret',
    },
    null,
    {
      store: sessionStore,
      resave: true,
      saveUninitialized: true,
      secret: 'sessionsecret',
      cookie: {
        httpOnly: process.env.NODE_ENV === 'production',
        secure: process.env.NODE_ENV === 'production',
      },
      name: 'adminjs',
    }
  )
  app.use(admin.options.rootPath, adminRouter)

  app.listen(PORT, () => {
    console.log(`AdminJS started on http://localhost:${PORT}${admin.options.rootPath}`)
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
$ yarn add connect-pg-simple
$ yarn add -D @types/connect-pg-simple @types/express-session @types/express
```

{% code title="app.ts" %}
```typescript
import AdminJS from 'adminjs'
import AdminJSExpress from '@adminjs/express'
import express from 'express'
import Connect from 'connect-pg-simple'
import session from 'express-session'

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
  const app = express()

  const admin = new AdminJS({})

  const ConnectSession = Connect(session)
  const sessionStore = new ConnectSession({
    conObject: {
      connectionString: 'postgres://adminjs:@localhost:5432/adminjs',
      ssl: process.env.NODE_ENV === 'production',
    },
    tableName: 'session',
    createTableIfMissing: true,
  })

  const adminRouter = AdminJSExpress.buildAuthenticatedRouter(
    admin,
    {
      authenticate,
      cookieName: 'adminjs',
      cookiePassword: 'sessionsecret',
    },
    null,
    {
      store: sessionStore,
      resave: true,
      saveUninitialized: true,
      secret: 'sessionsecret',
      cookie: {
        httpOnly: process.env.NODE_ENV === 'production',
        secure: process.env.NODE_ENV === 'production',
      },
      name: 'adminjs',
    }
  )
  app.use(admin.options.rootPath, adminRouter)

  app.listen(PORT, () => {
    console.log(`AdminJS started on http://localhost:${PORT}${admin.options.rootPath}`)
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

{% hint style="info" %}
For a more complex example, please visit our [example app](https://github.com/SoftwareBrothers/adminjs-example-app/blob/master/src/servers/express/index.ts).
{% endhint %}
