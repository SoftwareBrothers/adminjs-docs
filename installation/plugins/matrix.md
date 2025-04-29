---
description: >-
  This plugin integrates AdminJS with Matrix, enabling users to log in to the
  AdminJS panel using Matrix accounts through the MatrixAuthProvider. Users must
  have administrative privileges in Matrix to u
---

# Matrix

[➡️  Authentication - Matrix User Authentication](../../basics/authentication/matrixauthprovider.md)

***

### Features

* Seamless integration between AdminJS and Matrix.
* Authentication via Matrix user accounts.
* Resource management for Matrix users, rooms, and devices.

### Installation

Install the package using npm:

```bash
npm install @adminjs/matrix
```

### Environment Variables

Configure the following environment variables, preferably in a `.env` file:

```env
MATRIX_BASE_URL=http://localhost:8001
MATRIX_SERVER_DOMAIN=hhelocal.com
```

### Setup

#### 1. Import `reflect-metadata`

Import at the very beginning of your application’s entry file (e.g., `app.ts`):

```typescript
import 'reflect-metadata';
import express from 'express';
```

#### 2. Create Matrix Resources

Create resource files in `src/resources/matrix/`.

Example of general resource configuration:

```typescript
{
  sort: { sortBy: 'createdAt', direction: 'desc' },
  navigation: { name: 'Matrix', icon: 'MessageSquare' },
  actions: {
    new: { before: [] },
  },
  editProperties: ['name', 'password', 'displayname', 'userType', 'emailAddress', 'isAdmin', 'avatarUrl'],
  properties: {
    emailAddress: {
      type: 'string',
      isVisible: { list: false, filter: true, show: true, edit: true },
    },
  },
  translations: {
    en: {
      actions: { list: 'Matrix Users' },
      properties: {
        threepids: 'Addresses',
        'threepids.medium': 'Type',
      },
    },
  },
}
```

**matrix-device.resource.ts**

```typescript
import { DeviceAdapter, MatrixResourceEnum } from '@adminjs/matrix';

export const createMatrixDeviceResource = DeviceAdapter.buildMatrixDeviceResource({
  resourceType: MatrixResourceEnum.MATRIX_DEVICE,
  baseUrl: process.env.MATRIX_BASE_URL,
  serverDomain: process.env.MATRIX_SERVER_DOMAIN,
  eraseOnDelete: false,
});
```

**matrix-room.resource.ts**

```typescript
import { MatrixResourceEnum, RoomAdapter } from '@adminjs/matrix';

export const createMatrixRoomResource = RoomAdapter.buildMatrixRoomResource({
  resourceType: MatrixResourceEnum.MATRIX_ROOM,
  baseUrl: process.env.MATRIX_BASE_URL,
  serverDomain: process.env.MATRIX_SERVER_DOMAIN,
});
```

**matrix-user.resource.ts**

```typescript
import { MatrixResourceEnum, UserAdapter } from '@adminjs/matrix';

export const createMatrixUserResource = UserAdapter.buildMatrixUserResource({
  resourceType: MatrixResourceEnum.MATRIX_USER,
  baseUrl: process.env.MATRIX_BASE_URL,
  serverDomain: process.env.MATRIX_SERVER_DOMAIN,
  eraseOnDelete: false,
});
```

#### 3. Register Resources in AdminJS

Example `src/admin/options.ts`:

```typescript
import AdminJS, { AdminJSOptions } from 'adminjs';
import { DeviceAdapter, UserAdapter, RoomAdapter, matrixAuthEnTranslations } from '@adminjs/matrix'; 
import { createMatrixUserResource } from '../resources/matrix/matrix-user.resource.js';
import { createMatrixRoomResource } from '../resources/matrix/matrix-room.resource.js';
import { createMatrixDeviceResource } from '../resources/matrix/matrix-device.resource.js';
import componentLoader from './component-loader.js';

const adapters = [
  { Resource: UserAdapter.MatrixUserResource, Database: UserAdapter.MatrixUserDatabase },
  { Resource: RoomAdapter.MatrixRoomResource, Database: RoomAdapter.MatrixRoomDatabase },
  { Resource: DeviceAdapter.MatrixDeviceResource, Database: DeviceAdapter.MatrixDeviceDatabase },
];

adapters.forEach((adapter) => AdminJS.registerAdapter(adapter));

const options: AdminJSOptions = {
  componentLoader,
  rootPath: '/admin',
  resources: [
    createMatrixUserResource,
    createMatrixRoomResource,
    createMatrixDeviceResource,
  ],
  locale: {
    language: 'en',
    translations: {
      en: {
        properties: {
          ...matrixAuthEnTranslations.properties,
        },
        messages: {
          ...matrixAuthEnTranslations.messages,
        },
      },
    },
  },
};

export default options;
```

#### 4. Register Components

In `src/admin/component-loader.ts`:

```typescript
import { bundleMatrixAdapterComponents } from './addons/matrix/adapters/index.js';

bundleMatrixAdapterComponents(componentLoader);
```

### Authentication Methods <a href="#authentication-methods" id="authentication-methods"></a>

There are three ways to handle authentication with this plugin:

#### Method 1: Token-based Authentication <a href="#method-1-token-based-authentication" id="method-1-token-based-authentication"></a>

Add the Matrix token to your authentication provider (`src/admin/auth-provider.ts`):

```typescript
import { DefaultAuthProvider } from 'adminjs';
import componentLoader from './component-loader.js';
import { DEFAULT_ADMIN } from './constants.js';

const provider = new DefaultAuthProvider({
  componentLoader,
  authenticate: async ({ email }) => {
    if (email === DEFAULT_ADMIN.email) {
      return {
        email,
        _auth: {
          matrixAccessToken: process.env.MATRIX_ACCESS_TOKEN,
        },
      };
    }
    return null;
  },
});

export default provider;
```

Obtain your Matrix token by logging into Matrix. Execute this command on a machine that has access to the Matrix server:

```bash
curl --location 'http://localhost:8001/_matrix/client/r0/login' --header 'Content-Type: application/json' --data-raw '{ "type": "m.login.password", "user": "your_username", "password": "your_password" }'
```

Ensure the Matrix user is already registered and has administrative privileges. This command should be executed directly on the Matrix server as it's a built-in Matrix-Synapse tool:

```bash
register_new_matrix_user -c /path/to/matrix-synapse/homeserver.yaml http://localhost:8008
```

Note: The path to the homeserver.yaml file may vary depending on your Matrix installation. Common locations include:

/etc/matrix-synapse/homeserver.yaml /data/homeserver.yaml (if using Docker) Note: This method uses a single shared token for all AdminJS users, which remains valid indefinitely unless manually revoked.

#### Method 2: Matrix User Authentication <a href="#method-2-matrix-user-authentication" id="method-2-matrix-user-authentication"></a>

[Authentication - Matrix User Authentication](../../basics/authentication/matrixauthprovider.md)\


* Using this authentication method, ensure that the user's account has administrative privileges in Matrix.

#### Method 3: Custom Authentication Provider <a href="#method-3-custom-authentication-provider" id="method-3-custom-authentication-provider"></a>

You can also create a custom provider and manually handle tokens. Here's an example implementation of a custom authentication provider based on email and password:

```typescript
import { DefaultAuthProvider } from 'adminjs';
import componentLoader from './component-loader.js';

const provider = new DefaultAuthProvider({
  componentLoader,
  authenticate: async ({ email, password }) => {
    // Example implementation - replace with your own authentication logic
    const user = await db.users.findOne({ email });

    if (user && await verifyPassword(user.password, password)) {
      // Get or generate a Matrix token for this user
      const matrixToken = await getMatrixTokenForUser(user);

      return {
        email: user.email,
        _auth: {
          matrixAccessToken: matrixToken
        }
      };
    }

    return null;
  }
});

export default provider;
```

\
Contributing

Contributions to the `@adminjs/matrix` plugin are welcome. Please submit pull requests or open issues on GitHub for bug fixes, improvements, or new features.

### License

This plugin is licensed under the MIT License.
