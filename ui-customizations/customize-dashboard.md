# Customize dashboard

{% hint style="warning" %}
This article is outdated and it may contain information that is no longer up-to-date. It will be re-written soon!
{% endhint %}

By default AdminJS comes with the simple dashboard. You can easily modify it by adding some widgets.

### How to change default dashboard?

You can pass your own dashboard class to the AdminJS via [options](https://adminjs.page.link/options-interface)

```typescript
const DashboardPage = require('./dashboard-page')

const adminJsOptions = {
  ...
  databases: [...],
  resources: [...],
  dashboard: {
    handler: async () => {

    },
    component: AdminJS.bundle('./my-dashboard-component')
  },
  rootPath: '/admin'
  ...
}
```
