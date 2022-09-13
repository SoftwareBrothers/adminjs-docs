---
description: '@adminjs/objection'
---

# Objection

{% hint style="info" %}
Before reading this article, make sure you have set up an AdminJS instance using one of the supported [Plugins](../plugins/).\
Additionally, you should have installed `@adminjs/objection` as described in [Getting started](../getting-started.md) section.
{% endhint %}

This guide will assume you have set up Objection using it's [documentation](https://vincit.github.io/objection.js/guide/).

Before you start connecting your Objection models to AdminJS, we strongly advise to extend it's `Model` class to include format options and hooks for setting up your timestamps.

You might need to install an additional library `ajv-formats`:

```bash
$ yarn add ajv-formats
```

{% code title="base-model.ts" %}
```typescript
import addFormats from 'ajv-formats';
import { AjvValidator, Model } from 'objection';

export abstract class BaseModel extends Model {
  createdAt: string;

  updatedAt: string;

  static createValidator(): AjvValidator {
    return new AjvValidator({
      onCreateAjv: (ajv) => {
        addFormats(ajv);
      },
      options: {
        allErrors: true,
        validateSchema: false,
        ownProperties: true,
      },
    });
  }

  $beforeInsert(): void {
    this.createdAt = new Date().toISOString();
  }

  $beforeUpdate(): void {
    this.updatedAt = new Date().toISOString();
  }
}

```
{% endcode %}

`Office` is an example model which extends `BaseModel`:

{% code title="office.entity.ts" %}
```typescript
import { BaseModel } from '../base-model';

import Manager from './manager.entity';

export interface OfficeAddress {
  street: string;
  city: string;
  zipCode: string;
}

class Office extends BaseModel {
  id: number;

  name: string;

  address?: OfficeAddress;

  static tableName = 'offices';

  static jsonSchema = {
    type: 'object',
    required: ['name'],
    properties: {
      id: { type: 'integer' },
      name: { type: 'string', minLength: 1, maxLength: 255 },
      address: {
        type: 'object',
        properties: {
          street: { type: 'string' },
          city: { type: 'string' },
          zipCode: { type: 'string' },
        },
      },
      createdAt: { type: 'string', format: 'date-time' },
      updatedAt: { type: 'string', format: 'date-time' },
    },
  };
}

export default Office;
```
{% endcode %}

Please note that `jsonSchema` is necessary for AdminJS to determine your fields and their types.

The rest of the setup is similar to other adapters:

* You must import `AdminJSObjection` adapter and register it
* You must import the entities you want to use and pass them to AdminJS `resources` options, you cannot use `databases` because unfortunately Objection does not expose any connection with all models metadata.

{% code title="app.ts" %}
```typescript
// ... other imports
import * as AdminJSObjection from '@adminjs/objection'
import { Office } from './office.entity'

AdminJS.registerAdapter({
  Resource: AdminJSObjection.Resource,
  Database: AdminJSObjection.Database,
})

// ... other code
const start = async () => {
  const adminOptions = {
    // We pass Office to `resources`
    resources: [Office],
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

{% hint style="warning" %}
Make sure you have followed Objection documentation and have setup your `knexfile` and `knex` instance.
{% endhint %}

### Nest.js Support

A guide for Nest.js is a work in progress. If you would like to contribute to the documentation, please contact us.
