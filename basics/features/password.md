---
description: '@adminjs/passwords'
---

# Password

The password feature can be utilized to hash a user's password when editing its record.

## Installation

```shell
$ yarn add @adminjs/passwords
```

Next step is to add the feature option to the user resource:

```javascript
import argon2 from 'argon2';
import passwordsFeature from '@adminjs/passwords';

import User from './models/user.js';
import componentLoader from './component-loader.js';

const adminJsOptions = {
  resources: [
    {
      resource: User,
      options: {
        //...your regular options go here
        properties: { password: { isVisible: false } },
      },
      features: [
        passwordsFeature({
          componentLoader,
          properties: {
            encryptedPassword: 'password',
            password: 'newPassword',
          },
          hash: argon2.hash,
        })
      ]
    },
  ],
  //...
};
```

In the example above, `password` is the `User` property which holds the encrypted password (we have to make it invisible to keep it secure). `newPassword` in `passwordsFeature` -> `properties` is a virtual field which keeps the entered password and will be hashed before saving.
