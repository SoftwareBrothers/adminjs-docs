# Dashboard customization

By default, AdminJS comes with a simple dashboard which you may want to customize in most cases.

<figure><img src="../.gitbook/assets/Screenshot 2022-12-01 at 09.21.44.png" alt=""><figcaption><p>Default Dashboard</p></figcaption></figure>

The customization of the dashboard consists of two required and one optional steps:

1. Creating a React component for your dashboard
2. Configuring the dashboard to use your component
3. (Optional) Creating a `handler` for your dashboard to access server data

## Creating a React component for your dashboard

This step requires you to create a React component which is pretty much straightforward. Please refer to the [source code of default dashboard](https://github.com/SoftwareBrothers/adminjs/blob/master/src/frontend/components/app/default-dashboard.tsx) to get started.

`useTranslation` hook can be imported from `adminjs` library:

```typescript
import { useTranslation } from 'adminjs'
```

The rest of the code can be copied as is and worked on.

## Configuring the dashboard to use your component

Now you will have to tell AdminJS to use your dashboard. This can be achieved by configuring the `dashboard` option when instantiating `AdminJS`. Please refer to ["Writing your own Components"](writing-your-own-components.md) tutorial to find out how you can bundle your dashboard component.

Assuming that this is your `ComponentLoader`:

```typescript
import { ComponentLoader } from 'adminjs'

const componentLoader = new ComponentLoader()

const Components = {
  Dashboard: componentLoader.add('Dashboard', './dashboard'),
  // other custom components
}
```

This would be how you override the dashboard:

```typescript
const admin = new AdminJS({
  dashboard: {
    component: Components.Dashboard,
  },
  componentLoader
})
```

Restart your server and you should see your new, customized dashboard.

## (Optional) Creating a handler for your dashboard to access server data

In some cases you might want to access backend data in your dashboard, for example when you would like to display charts or statistics in general. To do this, you have to create a `handler` for you dashboard.

```typescript
const dashboardHandler = async () => {
  // Asynchronous code where you, e. g. fetch data from your database
  
  return { message: 'Hello World' }
}
```

The handler your create has to be assigned to `dashboard` option in your `AdminJS` instance:

```typescript
const admin = new AdminJS({
  dashboard: {
    component: Components.Dashboard,
    handler: dashboardHandler,
  },
  componentLoader
})
```

You now have the logic for returning the data to the frontend, the last part is accessing it in your dashboard component. To do this, you can use AdminJS's `ApiClient`  in your React component:

```tsx
import { ApiClient } from 'adminjs'
import React, { useEffect, useState } from 'react'

// ...
const [data, setData] = useState(null)
const api = new ApiClient()

useEffect(() => {
  api.getDashboard()
    .then((response) => {
      setData(response.data) // { message: 'Hello World' }
    })
    .catch((error) => {
      // handle any errors
    });
}, [])

// ...

console.log(data.message) // "Hello World"
```

Save your component, restart your server and you're good to go.
