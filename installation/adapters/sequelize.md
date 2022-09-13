---
description: '@adminjs/sequelize'
cover: ../../.gitbook/assets/sequelize.png
coverY: 0
---

# Sequelize

{% hint style="info" %}
Before reading this article, make sure you have set up an AdminJS instance using one of the supported [Plugins](../plugins/).\
Additionally, you should have installed `@adminjs/sequelize` as described in [Getting started](../getting-started.md) section.
{% endhint %}

This guide will assume you have set up Sequelize using it's [documentation](https://sequelize.org/docs/v6/getting-started/) or [Nest.js documentation](https://docs.nestjs.com/recipes/sql-sequelize).

There are small differences in how you connect Sequelize to Nest.js vs other plugins, so the guide will be split into two sections accordingly.

Example model:

{% code title="category.entity.ts" %}
```typescript
import { DataTypes, Model, Optional } from 'sequelize'
import sequelize from './index'

interface ICategory {
  id: number;
  name: string;
  createdAt: Date;
  updatedAt: Date;
}

export type CategoryCreationAttributes = Optional<ICategory, 'id'>

export class Category extends Model<ICategory, CategoryCreationAttributes> {
  declare id: number;
  declare name: string;
  declare createdAt: Date;
  declare updatedAt: Date;
}

Category.init(
  {
    id: {
      type: DataTypes.INTEGER,
      autoIncrement: true,
      primaryKey: true,
    },
    name: {
      type: new DataTypes.STRING(128),
      allowNull: false,
    },
    createdAt: {
      type: DataTypes.DATE,
    },
    updatedAt: {
      type: DataTypes.DATE,
    },
  },
  {
    sequelize,
    tableName: 'categories',
    modelName: 'category',
  }
)
```
{% endcode %}

Sequelize connection:

```typescript
import { Sequelize } from 'sequelize'

const sequelize = new Sequelize('postgres://adminjs:adminjs@localhost:5435/adminjs', {
  dialect: 'postgres',
})

export default sequelize
```

### Standard

Make sure you have followed the tutorial for the framework you are using in the [Plugins](../plugins/) section.

The configuration for non-Nest.js plugins is basically the same for each one of them:

* You must import `AdminJSSequelize` adapter and register it
* You must import the entities you want to use and pass them to AdminJS `resources` options

{% code title="app.ts" %}
```typescript
// ... other imports
import * as AdminJSSequelize from '@adminjs/sequelize'
import { Category } from './category.entity'

AdminJS.registerAdapter({
  Resource: AdminJSSequelize.Resource,
  Database: AdminJSSequelize.Database,
})

// ... other code
const start = async () => {
  const adminOptions = {
    // We pass Category to `resources`
    resources: [Category],
  }
  // Please note that some plugins don't need you to create AdminJS instance manually,
  // instead you would just pass `adminOptions` into the plugin directly,
  // an example would be "@adminjs/hapi"
  const admin = new AdminJS(adminOptions)
  // ... other code
}

start()
```
{% endcode %}

### Nest.js

Make sure you have set up your `app.module.ts` according to [Nest.js documentation](https://docs.nestjs.com/recipes/sql-typeorm) and you have followed [Nest.js plugin tutorial ](../plugins/nest.md)as well.

Your `app.module.ts` should have `imports` option which contains:

* `SequelizeModule.forRoot({ uri: '...', dialect: '...' })` to set up Sequelize
* `AdminModule.createAdminAsync({ ... }`

{% hint style="info" %}
You should also be able to get Sequelize to work if you set it up using a Nest.js database provider. The main point is to have a database connection established before AdminJS is initialized.
{% endhint %}

In your `app.module.ts` add these imports at the top of the file:

{% code title="app.module.ts" %}
```typescript
import * as AdminJSSequelize from '@adminjs/sequelize'
import AdminJS from 'adminjs'
```
{% endcode %}

Following this, register `AdminJSSequelize` adapter somewhere after your imports:

{% code title="app.module.ts" %}
```typescript
AdminJS.registerAdapter({
  Resource: AdminJSSequelize.Resource,
  Database: AdminJSSequelize.Database,
})
```
{% endcode %}

This will allow you to pass Sequelize models for AdminJS to load. If we use the `Category` entity that we used as en example earlier, you should import it into `app.module.ts` and pass it into `resources` in your `adminJsOptions`:

{% code title="app.module.ts" %}
```typescript
// ... other imports
import { Category } from './category.entity'
// ... other code
AdminModule.createAdminAsync({
  useFactory: () => ({
    adminJsOptions: {
      rootPath: '/admin',
      resources: [Category],
    },
  }),
}),
// ... other code
```
{% endcode %}
