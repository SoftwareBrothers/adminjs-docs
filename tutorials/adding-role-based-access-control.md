# Role-Based Access Control

Role-based access control allows your application to limit access to resources, records and actions only to specific users. This is not a feature in AdminJS, but rather a way of configuring it to your needs using your own, custom code.

## Setup

Your app should be set up with a [plugin](../installation/plugins/) and an [adapter](../installation/adapters/) with the authenticated router. This will give us access to the user object throughout the AdminJS config file. Let's assume your user object will look similar to this TypeORM model:

```typescript
@Entity({ name: 'users' })
class User extends BaseEntity {
  @PrimaryGeneratedColumn()
  public id!: number;

  @Column()
  public email!: string;

  @Column()
  public role!: string;

  @Column()
  public password!: string;
}
```

When setting up an authenticated router, there's an async `authenticate` function in the config. It receives email and password, and should return the matching user object like the one above (or `null` if credentials are invalid).

## Managing users

All passwords should be hashed in the database and never exposed to the outside world. AdminJS however doesn't know which fields of which models should be treated as such, so you need to manually remove passwords from the responses.

This is done using `after` and `before` hooks - you can inspect the request and response objects and modify them before or after they're handled in AdminJS by removing or hashing all passwords. Note that `after` hook in the `edit` action is called twice - first time as `GET` to get the data for editing (we need to clear passwords there) and again as `POST` to update the data (here we need to hash the new password). Also, `list` action returns a _list_ of records, so we need to iterate over them to remove passwords. This makes most action hooks a little bit different.

Lastly, we can hide password fields from views that don't need to display them. This is done using `isVisible` setting in properties. Please remember that this only hides the UI elements, it doesn't prevent AdminJS from sending the property, so we still need those `after` hooks.

```typescript
const userResource: ResourceWithOptions = {
  resource: User,
  options: {
    actions: {
      new: {
        before: async (request) => {
          if (request.payload?.password) {
            request.payload.password = hash(request.payload.password);
          }
          return request;
        },
      },
      show: {
        after: async (response: RecordActionResponse) => {
          response.record.params.password = '';
          return response;
        },
      },
      edit: {
        before: async (request) => {
          // no need to hash on GET requests, we'll remove passwords there anyway
          if (request.method === 'post') {
            // hash only if password is present, delete otherwise
            // so we don't overwrite it
            if (request.payload?.password) {
              request.payload.password = hash(request.payload.password);
            } else {
              delete request.payload?.password;
            }
          }
          return request;
        },
        after: async (response: RecordActionResponse) => {
          response.record.params.password = '';
          return response;
        },
      },
      list: {
        after: async (response: ListActionResponse) => {
          response.records.forEach((record) => {
            record.params.password = '';
          });
          return response;
        },
      },
    },
    properties: {
      password: {
        isVisible: {
          list: false,
          filter: false,
          show: false,
          edit: true, // we only show it in the edit view
        },
      },
    },
  },
};
```

You can find more information about adding logic to your properties in the Property Logic tutorial.

## Restricting access to actions

To remove access to a whole action you can use `isAccessible` setting. This will remove the UI elements and block all API requests for those specific actions.

```typescript
const someResource: ResourceWithOptions = {
  resource: Something,
  options: {
    actions: {
      new: {
        isAccessible: false,
      },
    },
  },
};
```

You can also pass it a function that decides whether currently performed action is accessible depending on context like current user:

```typescript
const someResource: ResourceWithOptions = {
  resource: Something,
  options: {
    actions: {
      new: {
        isAccessible: ({ currentAdmin }) => currentAdmin.role === 'admin',
      },
    },
  },
};
```

Or restrict access to specific actions based on the content of the record. All this applies to custom actions as well.

```typescript
const someResource: ResourceWithOptions = {
  resource: Something,
  options: {
    actions: {
      publish: {
        isAccessible: ({ record }) => !record.params.published,s
        // rest of the custom action code
      },
    },
  },
};
```

### Difference between `isAccessible` and `isVisible`

Both of those settings hide specific actions from the UI, but `isAccessible` also blocks API calls to this action. If you have custom components where you want to programatically call API endpoints for specific action, but don't want to display it to the user, you can use `isVisible` to hide it instead. This is often useful for custom actions that have some additional logic behind triggering them.

## Restricting access to specific properties

By default, AdminJS displays and allows editing of all properties of an object. You can control that to a degree using options `listProperties`, `editProperties` etc, but this hides the UI for all users. Hiding these properties based on user's role is a little bit more involved.

In general, you want to capture the `resource` object before it's rendered by an action component, and remove all properties that shouldn't be displayed for this specific role. You do this by creating a custom action component that adjusts the data and passes it back into the default action component.

```typescript
import React, { FC } from 'react';
import {
  ActionProps,
  BaseActionComponent,
  BasePropertyJSON,
  useCurrentAdmin,
} from 'adminjs';

const CustomAction: FC<ActionProps> = (props) => {
  const [currentAdmin] = useCurrentAdmin();
  const newProps = { ...props };
  
  // This is important - `component` option controls which custom
  // component is rendered by `BaseActionComponent` and we don't
  // want to render this code here again. That would create an
  // infinite loop.
  newProps.action = { ...newProps.action, component: undefined };

  // Configuration is stored in each property's custom props.
  const filter = (property: BasePropertyJSON) => {
    const { role } = property.custom;
    return !role || currentAdmin?.role === String(role);
  };

  // Since we want to remove properties from all actions, a common
  // filtering function can be used.
  const { resource } = newProps;
  resource.listProperties = resource.listProperties.filter(filter);
  resource.editProperties = resource.editProperties.filter(filter);
  resource.showProperties = resource.showProperties.filter(filter);
  resource.filterProperties = resource.filterProperties.filter(filter);

  // `BaseActionComponent` will now render the default action component
  return <BaseActionComponent {...newProps} />;
};

export default CustomAction;
```

The code above will hide the UI elements on the frontend, but the responses from the API will still contain all data that was hidden. Editing hidden data with tools like Postman will also be possible. You may want to patch that up using action hooks. Here's an example of an `after` hook that is generalized for every action:

```typescript
const roleAccessControlAfterHook = async (
  response: any,
  _: any,
  context: ActionContext,
) => {
  const { properties } = context.resource
    .decorate()
    .toJSON(context.currentAdmin);
  const targetRole = context.currentAdmin?.role;
  const propertiesToRemove = Object.entries(properties)
    .filter(
      ([_, { custom }]) => custom.role && String(custom.role) !== targetRole,
    )
    .map(([name]) => name);

  const cleanupRecord = (record: RecordJSON) => {
    propertiesToRemove.forEach((name) => delete record.params[name]);
  };
  if (response.record) {
    cleanupRecord(response.record);
  }
  if (response.records) {
    response.records.forEach(cleanupRecord);
  }
  return response;
};
```

Similar thing can be implemented for `POST` request in `before` hooks in `edit` and `new` actions in order to prevent editing those fields.

```typescript
const roleAccessControlBeforeHook: Before = async (request, context) => {
  const { method, payload } = request;
  if (method !== 'post' || !payload) {
    return request;
  }
  const { properties } = context.resource
    .decorate()
    .toJSON(context.currentAdmin);
  const targetRole = context.currentAdmin?.role;
  const propertiesToRemove = Object.entries(properties)
    .filter(
      ([_, { custom }]) => custom.role && String(custom.role) !== targetRole,
    )
    .map(([name]) => name);
  propertiesToRemove.forEach((name) => delete payload[name]);
  return request;
};
```

In case of the `new` action you might want to add additional `before` hook that would set a default value for the fields you're restricting if they are required in the database.

```typescript
const defaultValuesBeforeHook: Before = async (request, context) => {
  const { payload, method } = request;
  if (method !== 'post' || !payload || context.action.name !== 'new') {
    return request;
  }
  const { properties } = context.resource
    .decorate()
    .toJSON(context.currentAdmin);
  Object.entries(properties).forEach(([name, { custom }]) => {
    if (custom.defaultValue && payload[name] === undefined) {
      payload[name] = custom.defaultValue;
    }
  });
  return request;
};
```

If you need to reuse this functionality in other resources, it might be a good idea to pack it into a feature:

```typescript
const roleBasedAccessControl = buildFeature((admin) => {
  const CustomAction = admin.componentLoader.add(
    'CustomAction',
    './custom-action',
  );
  return {
    actions: {
      new: {
        component: CustomAction,
        before: [roleAccessControlBeforeHook, defaultValuesBeforeHook],
        after: [roleAccessControlAfterHook],
      },
      edit: {
        component: CustomAction,
        before: [roleAccessControlBeforeHook],
        after: [roleAccessControlAfterHook],
      },
      show: {
        component: CustomAction,
        after: [roleAccessControlAfterHook],
      },
      list: {
        component: CustomAction,
        after: [roleAccessControlAfterHook],
      },
    },
  };
});
```

Finally, this configuration will hide the properties specified in the action config from users without a specific role:

```typescript
const someResource: ResourceWithOptions = {
  resource: Something,
  features: [roleBasedAccessControl],
  options: {
    properties: {
      superSecretAdminProperty: {
        custom: {
          role: 'admin',
          defaultValue: 'a secret',
        },
      },
    },
  },
};
```
