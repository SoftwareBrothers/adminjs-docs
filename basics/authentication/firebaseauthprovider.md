---
description: '@adminjs/firebase-auth'
---

# FirebaseAuthProvider

`@adminjs/firebase-auth` is an authentication provider which allows you to sign in using Firebase Authentication.



<figure><img src="../../.gitbook/assets/Screenshot 2023-11-22 at 13.25.33.png" alt=""><figcaption></figcaption></figure>

## Prerequisites

Make sure you follow official Firebase documentation to properly set up Firebase Authentication in Firebase Console: [https://firebase.google.com/docs/auth](https://firebase.google.com/docs/auth)

## Installation

`@adminjs/firebase-auth` is a premium feature which can be purchased at [https://cloud.adminjs.co](https://cloud.adminjs.co)

All premium features currently use **One Time Payment** model and you can use them in all apps that belong to you. Once you purchase the addon, you will receive a license key which you should provide in `@adminjs/firebase-auth` configuration in your application's code.

Installing the library:

```bash
$ yarn add @adminjs/firebase-auth
```

The license key should be provided via `FirebaseAuthProvider` constructor:

```typescript
new FirebaseAuthProvider({
  licenseKey: process.env.LICENSE_KEY,
  // the rest of the config
})
```

If you encounter any issues or require help installing the package please contact us through our Discord server.

## Usage

`FirebaseAuthProvider` requires you to prepare Firebase-specific configuration.

#### UI Config

`@adminjs/firebase-auth` uses `firebaseui-web` to generate Firebase UI inside the login form and it requires you to configure it. Please refer to the following link for configuration options: [https://github.com/firebase/firebaseui-web/blob/master/types/index.d.ts#L115](https://github.com/firebase/firebaseui-web/blob/master/types/index.d.ts#L115)

Note that you can only configure raw values but you will not be able to configure functions or callbacks this way. A workaround will be described in the later part of this documentation.

```typescript
import { EmailAuthProvider } from 'firebase/auth';

const uiConfig = {
  popupMode: true,
  signInFlow: 'popup',
  signInOptions: [
    {
      provider: EmailAuthProvider.PROVIDER_ID,
      disableSignUp: {
        status: true,
      },
    },
  ],
};
```

#### Firebase Configuration

`@adminjs/firebase-auth` initializes a Firebase App in the Login view. Make sure you copy the configuration from your project in Firebase Console.

```typescript
const firebaseConfig = {
  apiKey: 'AIza...',
  authDomain: 'XXXX.firebaseapp.com',
  projectId: 'XXXX',
  storageBucket: 'XXXX.appspot.com',
  messagingSenderId: '11111111111',
  appId: '1:11111111111:web:abcdef',
};
```

You will most likely also have to initialize a Firebase app on your server's end to verify the user's access token later:

```typescript
import { initializeApp } from 'firebase-admin/app';

// ...

const firebaseApp = initializeApp(firebaseConfig);
```

#### Authenticate method

Lastly, you must define an `authenticate` method which you will use to verify the user's access token and return the user object.

```typescript
import { getAuth } from 'firebase-admin/auth';
import { FirebaseAuthenticatePayload } from '@adminjs/firebase-auth';

export const authenticate = async ({
  accessToken,
}: FirebaseAuthenticatePayload) => {
  const auth = getAuth(firebaseApp);

  try {
    const decodedToken = await auth.verifyIdToken(accessToken);

    return {
      id: decodedToken.uid,
      email: decodedToken.email ?? '',
      avatarUrl: decodedToken.picture,
    };
  } catch (error) {
    console.log(error);
    return null;
  }
};
```

### FirebaseAuthProvider Configuration

After you are done with Firebase-specific configuration, you can instantiate `FirebaseAuthProvider`:

```typescript
import { FirebaseAuthProvider } from '@adminjs/firebase-auth';
import componentLoader from '<path to your component loader file>';

// ... assume Firebase related configuration is in the same file

const authProvider = new FirebaseAuthProvider({
  // make sure that the same ComponentLoader instance is configured in AdminJS!
  componentLoader,
  uiConfig,
  firebaseConfig,
  authenticate,
  licenseKey: process.env.LICENSE_KEY,
});

// ...

// Add "provider" to authentication options of your framework plugin, Express example:

const router = buildAuthenticatedRouter(
  admin,
  {
    cookiePassword: 'test',
    provider: authProvider,
  },
  null,
  {
    secret: 'test',
    resave: false,
    saveUninitialized: true,
  }
);
```

With your plugin and auth provider configured you should be able to restart your server and a new login page with embedded Firebase UI should appear.

## Troubleshooting

#### How to prevent @adminjs/firebase-auth from overriding the Login page?

By default, `@adminjs/firebase-auth` will override your Login page with it's own UI. You can disable that by adding `overrideLogin: false` in `FirebaseAuthProvider` configuration:

```typescript
const authProvider = new FirebaseAuthProvider({
  componentLoader,
  uiConfig,
  firebaseConfig,
  authenticate,
  licenseKey: process.env.LICENSE_KEY,
  overrideLogin: false,
});
```

If you do this, though, Firebase won't render it's own UI. You will have to create your own Login page and import `FirebaseAuthForm` component from `@adminjs/firebase-auth` and put it wherever you want in your own Login component.

```typescript
import { FirebaseAuthForm } from '@adminjs/firebase-auth/components'

const CustomLogin = () => {
 return <FirebaseAuthForm />
}

export default CustomLogin
```

Remember to override `Login` using your component loader:

```typescript
componentLoader.override('Login', '<path to custom login>');
```

#### How to configure functions and callbacks of UI Config?

Follow the steps above to create your custom Login page. Create UI configuration in your custom component and provide it as props of `FirebaseAuthForm`:

```typescript
const uiConfig = {/* custom config */};

const CustomLogin = () => {
 return <FirebaseAuthForm uiConfig={uiConfig} />
}
```

#### How to customize the look of Firebase UI?

If you decide to follow the steps above you will be able to create a styled component out of `FirebaseAuthForm` and customize it fully.

Alternatively, you can create a CSS file and add it to your app's assets.

```css
.adminjs_firebaseui-container {
  background: red;
}
```

Make sure the CSS file is present in your server's public assets directory. Lastly, configure `assets` of AdminJS:

```typescript
const admin = new AdminJS({
  assets: {
    styles: ['/firebase-ui.css'],
  },
  // other config
})
```
