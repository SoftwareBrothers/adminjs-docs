---
description: >-
  Resource is something that you can manage in AdminJS and it comes with CRUD
  actions (Create, Read, Update, Delete) provided out of the box.
---

# Resource

## Introduction

Resource is something that you can manage in AdminJS and it comes with CRUD actions _(Create, Read, Update, Delete)_ provided out of the box. Most of the time it is a model from your ORM or ODM.

The idea of AdminJS is to allow you to manage resources of all kinds, be it your ORM/ODM models or your custom REST API endpoints if you decide to create an [adapter ](../installation/adapters/)for them.

## Adapters

AdminJS allows you to define resources through adapters. Adapters are AdminJS extensions which provide the API to communicate with your ORM, ODM or any other kind of storage or API of your choice. All adapters must extend three base classes by implementing their methods:

* [BaseResource](https://adminjs.page.link/base-resource-code) - it's responsible for CRUD operations on every resource (model) that your provide
* [BaseDatabase](https://adminjs.page.link/base-database-code) - it's responsible for loading all resources (models) defined in your database (if you choose to do so)
* [BaseProperty](https://adminjs.page.link/base-property-code) - it's responsible to describe your resource's (model's) attributes based on your model's metadata

The list of adapters that are officially supported by AdminJS can be found in [Adapters](../installation/adapters/) section.

In order to use an adapter in your documentation you must first register it.

```typescript
import AdminJS from 'adminjs'
import { Database, Resource } from '@adminjs/typeorm' // or any other adapter

AdminJS.registerAdapter({ Database, Resource })
```

You can register as many adapters as you need.

## Passing resources to AdminJS

There are two options which you can choose to provide resources to AdminJS:

1. Provide entire database connection
2. Provide every resource explicitly

### Providing entire database

This option allows you to provide an entire database and AdminJS will load all models that you have defined. However, this option may not be available for every adapter. The adapter must expose a connection or client which exposes the metadata of all models that it had loaded. `@adminjs/objection` is an example of an adapter which **does not** allow you to provide an entire database.

In order to provide an entire database for AdminJS to load, you must specify `databases` property when setting up your AdminJS instance. In case of `@adminjs/mongoose` it would look as follows:

```typescript
const mongooseDb = await mongoose.connect('mongodb://localhost:27017/test')

const admin = new AdminJS({
  databases: [mongooseDb],
})
```

For other adapters the setup would be basically the same, you just have to pass your ORM/ODM connection (or a client).

### Providing resources explicitly

This option requires you to define all resources explicitly. It is also the recommended approach since it allows you to customize every resource. In order to define resources, you must specify `resources` property when setting up your AdminJS instance. Example:

```typescript
import User from './user.entity.js'
import Profile from './profile.entity.js'

// User and Profile are models defined in your ORM/ODM

const admin = new AdminJS({
  resources: [
    User, // you can simply pass a model
    {
      resource: Profile,
      options: { // or you can provide an object with your custom resource options
        id: 'profiles', // here the resource identifier has been renamed to "profiles"
      },
    },
  ],
})
```

The way you provide resources may differ between adapters. Make sure you read a detailed tutorial for an [adapter](../installation/adapters/) which you are using.

## Customizing resources

While AdminJS provides default CRUD for your application, you may want to further customize your resources. This can be done by using object definition of a resource and specifying the [ResourceOptions](https://adminjs.page.link/resource-options-code). The example above is the simplest possible where we change the `id` of a resource to `profiles`. Below you will find several examples of resource customization.

### Nesting a resource under collapsible navigation

This can be achieved by specifying `navigation` property in your resource's options, example:

```typescript

const usersNavigation = {
  name: 'Users',
  icon: 'User',
}

const admin = new AdminJS({
  resources: [{
    resource: Profile,
    options: {
      navigation: usersNavigation,
    },
  }],
})
```

This will put the `Profile` resource under `Users` menu.

### Changing visibility of properties

If you want specific properties be displayed or usable in a given action, there are two options to achieve this:

1. Set `isVisible` option for every property
2. Set `listProperties`, `editProperties`, `filterProperties`, `showProperties` in your resource

#### Setting \`isVisible\`

Every property that you have defined in your database model can be further customized in AdminJS. In this example we will hide `bio` property in `list` action and hide it from filters, but we will leave it enabled in `show` and `edit`:

```typescript
const admin = new AdminJS({
  resources: [{
    resource: Profile,
    options: {
      properties: {
        bio: {
          isVisible: {
            edit: true,
            show: true,
            list: false,
            filter: false,
          },
        },
      },
    },
  }],
})
```

#### Settings lists of visible properties

The example with `bio` hidden in `list` and `filter` but visible in `show` and `edit` can also be achieved by setting `listProperties`, `editProperties`, `filterProperties`, `showProperties`

```typescript
const admin = new AdminJS({
  resources: [{
    resource: Profile,
    options: {
      listProperties: ['id', 'name', 'createdAt'],
      filterProperties: ['id', 'name', 'createdAt'],
      editProperties: ['id', 'name', 'bio', 'createdAt'],
      showProperties: ['id', 'name', 'bio', 'createdAt'],
    },
  }],
})
```

The end result is the same but you should take note that this approach takes precedence over setting `isVisible` or property's `position`.

More examples of properties' customization can be found in [Property](property.md) section.

### Customizing actions

Please refer to [Action](action.md) section for examples and explanation.

### Changing default navigation link

By default, when you press a resource link in the sidebar, it will navigate to resource's `list` action. This can be changed by configuring `href` option. Let's say we want `users` resource to open an already-filtered `users` list which show only users that have "active" status:

```typescript
const UserResource = {
  resource: User,
  options: {
    id: 'users',
    href: ({ h, resource }) => {
      return h.resourceActionUrl({
        resourceId: resource.decorate().id(),
        actionName: 'list',
        params: {
          'filters.status': 'active',
        },
      })
    },
  },
}
```

### Configuring sorting for the resource

By default, when you navigate to your resource's `list` action it will show you results in random order since the actual database query will be lacking sorting information (unless you use table UI to select a column to sort by). You can, however, define default sorting for your resource by specifying `sort` option:

```typescript
const UserResource = {
  resource: User,
  options: {
    sort: {
      sortBy: 'updatedAt',
      direction: 'desc',
    },
  },
}
```

In the example above, we specified User resource to be sorted by it's `updatedAt` property with the latest records appearing at the top of the results list.

### Resource translations

You can define translations for your resources by specifying `locale`. This is not done on resource-level though, but during the instantiation process of AdminJS.&#x20;

#### Renaming a resource in sidebar

In order to rename your resource in the sidebar of the application, you have to set it's label:

```typescript
const admin = new AdminJS({
  resources: [User],
  locale: {
    language: 'en',
    translations: {
      labels: {
        User: 'People',
      },
    },
  },
})
```

In the example above, the `User` resource in the sidebar has been renamed to `People`.

Another scenario would be where you would want to have different messages shown based on which resource the user is currently viewing. An example would be a message which appears when you enter a resource without any records to show in `list` action:

> There are no records in this resource

In order to change it you have to define resource-specific translation as in the example below:

```typescript
const admin = new AdminJS({
  resources: [User],
  locale: {
    language: 'en',
    translations: {
      resources: {
        User: {
          messages: {
            noRecordsInResource: 'There are no users to display'
          },
        },
      },
    },
  },
})
```

To see a list of all available locales and predefined translations, please visit [locales](https://adminjs.page.link/locale-code) in `adminjs` core repository.

### Using "features"

Features are predefined pieces of code which you can import into your resource's `features`  and they will be merged with the rest of it's configuration. An example of a `feature` is `@adminjs/passwords`. This is a package which handles passwords hashing in `edit` form and shows a corresponding UI.

Usage example:

```typescript
import passwordsFeature from '@adminjs/passwords'
import argon2 from 'argon2'

import { User } from './user.entity.js'

const UserResource = {
  resource: User,
  features: [passwordsFeature({
    properties: { encryptedPassword: 'hashedPassword' },
    hash: argon2.hash,
  })],
}
```

