---
description: '@adminjs/relations'
---

# Relations

`@adminjs/relations` is a feature which allows you to manage `one-to-many` and `many-to-many` relations within your admin panel.

As of version `1.0.0` it supports:

* listing multiple `one-to-many` relations with pagination but no filters in the details view of a record,
* listing multiple `many-to-many` relations with pagination but no filters in the details view of a record,
* editing and creating `one-to-many` relations,
* editing `many-to-many` relations,
* deleting records listed in `one-to-many` table (if you want to only remove the relation, you can just modify the target record)
* deleting relations in junction table of `many-to-many` relation **or** deleting the target record of a `many-to-many` relation (it also deleted the relation in junction table)
* navigating to details view of a target relation,
* adding an existing record to a `many-to-many` relation or creating a new record which will be assigned to your `many-to-many` relation.

<div>

<figure><img src="../../.gitbook/assets/Screenshot 2023-04-26 at 12.34.31.png" alt=""><figcaption><p>One-To-Many List</p></figcaption></figure>

 

<figure><img src="../../.gitbook/assets/Screenshot 2023-04-26 at 12.36.20.png" alt=""><figcaption><p>Many-To-Many List</p></figcaption></figure>

 

<figure><img src="../../.gitbook/assets/Screenshot 2023-04-26 at 12.36.39.png" alt=""><figcaption><p>Many-To-Many Modal</p></figcaption></figure>

</div>

## Installation

`@adminjs/relations` is a premium feature which can be purchased at [https://cloud.adminjs.co](https://cloud.adminjs.co)

All premium features currently use **One Time Payment** model and you can use them in all apps that belong to you. Once you purchase the addon, you will receive a license key which you should provide in `@adminjs/relations` configuration in your application's code.

Installing the library:

```bash
$ yarn add @adminjs/relations
```

The license key should be provided to `owningRelationSettingsFeature`:

```typescript
owningRelationSettingsFeature({
  licenseKey: process.env.LICENSE_KEY,
  // the rest of the config
})
```

`targetRelationSettingsFeature` does not require a license key as it's role is mostly utility-only. The documentation below describes the configuration objects and setup instructions in more detail.

If you encounter any issues or require help installing the package please contact us at adminjs@adminjs.co or through our Discord server.

## Usage

Similarly to other features, the `@adminjs/relations` feature has to be imported into `features` configuration section of your resource. `@adminjs/relations` exports two separate feature that you must configure in order for the functionality to work:

* `owningRelationSettingsFeature` is used to configure the relations that you will want to manage later,
* `targetRelationSettingsFeature` doesn't require any configuration, but it has to be included in targetted resource in order for redirects and `many-to-many` assignments to work properly.

The two features will be explained in more detail in the later parts of this guide.

### Example Database Structure

The usage guide will be based on sample database tables which can be represented by the following interfaces:

```typescript
interface IOrganization {
  id: number;
  name: string;
}

interface ITeam {
  id: number;
  name: string;
}

interface IPerson {
  id: number;
  name: string;
  email: string;
  organizationId: number;
}

interface ITeamMember {
  id: number;
  personId: number;
  teamId: number;
}

interface IOffice {
  id: number;
  name: string;
  address: string;
  organizationId: string;
}

/*
  Person belongs to 1 Organization
  Organization has many Persons
  Office belongs to 1 Organization
  Organization has many Offices
  Person belongs to multiple Teams through TeamMember
  Team belongs to multiple Persons through TeamMember
*/
```

`@adminjs/relations` is adapter-agnostic which means you can use it regardless of the database adapter you had installed. Nevertheless, some ORMs automatically generate and manage  junction tables for you without you having to actually create entities for them in your codebase. This will not work with AdminJS and you will have to create actual entities for junction tables and register them as AdminJS resources since AdminJS uses them to find your `M:N` records.

```typescript
const admin = new AdminJS({
  resources: [
    createOrganizationResource(),
    createPersonResource(),
    createOfficeResource(),
    createTeamResource(),
    createTeamMemberResource(),
  ],
})
```

{% hint style="warning" %}
AdminJS requires every resource to have a primary key column, this includes junction tables.
{% endhint %}

### Feature Options

Below you can find feature options of `owningRelationSettingsFeature` which you can use for reference.

```typescript
enum RelationType {
  OneToMany = 'one-to-many',
  ManyToMany = 'many-to-many',
}

type RelationsFeatureConfig = {
  /* Your ComponentLoader instance, ideally you will create it in a separate file
  and import where it's needed. Documentation: https://docs.adminjs.co/ui-customization/writing-your-own-components */
  componentLoader: ComponentLoader;
  /* Your license key */
  licenseKey: string;
  /* A configuration object for relations that will be managable in a given resource. */
  relations: {
    /* A name of a relation. It will be used as a name in tabbed table (see screenshots above) */
    [resourceId: string]: {
      /* A relation type which can be either `one-to-many` or `many-to-many` */
      type: RelationType;
      /* A junction resource/table configuration. It is only required for `many-to-many` */
      junction?: {
        /* A "joinKey" inside junction table. If configuring for "Team", it can be "teamId". */
        joinKey: string;
        /* An "inverseJoinKey" inside junction table. If "Team" has a M:N relation with "Person", it can be "personId" */
        inverseJoinKey: string;
        /* A resource ID of the junction table, for example: "TeamMember" */
        throughResourceId: string;
      };
      /* A target resource/table configuration. A target is a resource which is listed in the table. */
      target: {
        /* A "resourceId" of the target. Example: "Person" */
        resourceId: string;
        /* A "joinKey" of the target. Example: "organizationId" */
        joinKey?: string;
      };
    }
  };
  /* An optional field which allows you to specify a different property key which will be used
  to display relations table. By default it adds `relations` to details view of your resource. */
  propertyKey?: string;
};
```

### One-To-Many

According to the database structure described above as well as the presented configuration options of `owningRelationSettingsFeature`, this is how you can add this feature to `Organization` resource which can have many `Persons` and `Offices`

{% code title="organization.resource.ts" %}
```typescript
import { owningRelationSettingsFeature, type RelationType } from '@adminjs/relations'
import { componentLoader } from './component-loader.js';
import { Organization } from './models/index.js';

export const createOrganizationResource = () => ({
  resource: Organization,
  features: [
    owningRelationSettingsFeature({
      componentLoader,
      licenseKey: process.env.LICENSE_KEY,
      relations: {
        persons: {
          type: RelationType.OneToMany,
          target: {
            joinKey: 'organizationId',
            resourceId: 'Person',
          },
        },
        offices: {
          type: RelationType.OneToMany,
          target: {
            joinKey: 'organizationId',
            resourceId: 'Office',
          },
        },
      },
    }),
  ],
});
```
{% endcode %}

Additionally, in your `Office` and `Person` resources you will have to add `targetRelationSettingsFeature`:

{% code title="office.resource.ts" %}
```typescript
import { targetRelationSettingsFeature } from '@adminjs/relations';
import { Office } from './models/index.js';

export const createOfficeResource = () => ({
  resource: Office,
  features: [targetRelationSettingsFeature()],
});
```
{% endcode %}

{% code title="person.resource.ts" %}
```typescript
import { targetRelationSettingsFeature } from '@adminjs/relations';
import { Person } from './models/index.js';

export const createPersonResource = () => ({
  resource: Person,
  features: [targetRelationSettingsFeature()],
});
```
{% endcode %}

If you configure your resources as shown above, you should be able to see `Persons` and `Offices` tabs in your `Organization` record's details view.

### Many-To-Many

The example below shows how you can configure a `many-to-many` relation between `Team` and `Person` through `TeamMember`.

{% code title="team.resource.ts" %}
```typescript
import { owningRelationSettingsFeature, type RelationType } from '@adminjs/relations';
import { Team } from './models/index.js';
import { componentLoader } from './component-loader.js';

export const createTeamResource = () => ({
  resource: Team,
  options: {
    navigation: { icon: 'Users' },
  },
  features: [
    owningRelationSettingsFeature({
      componentLoader,
      licenseKey: process.env.LICENSE_KEY,
      relations: {
        members: {
          type: RelationType.ManyToMany,
          junction: {
            joinKey: 'teamId',
            inverseJoinKey: 'personId',
            throughResourceId: 'TeamMember',
          },
          target: {
            resourceId: 'Person',
          },
        },
      },
    }),
  ],
});
```
{% endcode %}

Additionally, in your `Person` resource you will have to make sure to add `targetRelationSettingsFeature`. Of course, if you had added it before you don't have to add it multiple times.

{% code title="person.resource.ts" %}
```typescript
import { targetRelationSettingsFeature } from '@adminjs/relations';
import { Person } from './models/index.js';

export const createPersonResource = () => ({
  resource: Person,
  features: [targetRelationSettingsFeature()],
});
```
{% endcode %}

### Role Based Access Control

By default all actions related to managing the relations will be available for everyone. You can modify the accessibility in the same way you modify accessibility of your custom actions.

`@adminjs/relations` introduces the following new actions to your resource:

* `findRelation` is used to list `one-to-many` and `many-to-many` records from the target resource,
* `addManyToManyRelation` is used to add existing records to a junction table for `many-to-many` relations
* `deleteRelation` is used to delete a record from a junction table, deleting the relation in the process, but leaving both records

Taking `Team` resource from above as an example, you can allow these actions only for users with role `Admin` by doing the following changes:

```typescript
import { owningRelationSettingsFeature, type RelationType } from '@adminjs/relations';
import { Team } from './models/index.js';
import { componentLoader } from './component-loader.js';

const onlyForAdmin = ({ currentAdmin }) => currentAdmin.role === 'Admin';

export const createTeamResource = () => ({
  resource: Team,
  options: {
    navigation: { icon: 'Users' },
    actions: {
      findRelation: { isAccessible: onlyForAdmin },
      addManyToManyRelation: { isAccessible: onlyForAdmin },
      deleteRelation: { isAccessible: onlyForAdmin },
    },
  },
  features: [
    owningRelationSettingsFeature({
      componentLoader,
      licenseKey: process.env.LICENSE_KEY,
      relations: {
        members: {
          type: RelationType.ManyToMany,
          junction: {
            joinKey: 'teamId',
            inverseJoinKey: 'personId',
            throughResourceId: 'TeamMember',
          },
          target: {
            resourceId: 'Person',
          },
        },
      },
    }),
  ],
});
```

You can read more about RBAC in AdminJS in [this tutorial](../../tutorials/adding-role-based-access-control.md).
