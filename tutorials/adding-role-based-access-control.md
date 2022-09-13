# Adding Role-Based Access Control

{% hint style="warning" %}
This article is outdated and it may contain information that is no longer up-to-date. It will be re-written soon!
{% endhint %}

In this brief tutorial, I will present how you can add a role-based Admin Panel to your Node.js app. You can use this knowledge to build an entire application with access roles for managing different sort of data in 10 minutes.

ok — MVP version of that application :)

### The stack

These are the things we will use

* as a router, we will use an Express framework
* for persistent storage, we will use MongoDB with mongoose ODM
* and of course - adminjs

So let’s start!

### Setup the application

First of all, let’s create the folder and init new Node.js app there:

```
mkdir my-admin-app
cd my-admin-app
yarn init -y
```

Install the dependencies:

```
yarn add express express-formidable mongoose adminjs @adminjs/mongoose @adminjs/express
```

and copy this example app:

```typescript
// Requirements
const mongoose = require('mongoose')
const express = require('express')
const AdminJS = require('adminjs')
const AdminJSExpress = require('@adminjs/express')

// We have to tell AdminJS that we will manage mongoose resources with it
AdminJS.registerAdapter(require('@adminjs/mongoose'))

// express server definition
const app = express()

// Resources definitions
const User = mongoose.model('User', {
  email: { type: String, required: true },
  password: { type: String, required: true },
  role: { type: String, enum: ['admin', 'restricted'], required: true },
})

// Pass all configuration settings to AdminJS
const adminJs = new AdminJS({
  resources: [User],
  rootPath: '/admin',
})

// Build and use a router which will handle all AdminJS routes
const router = AdminJSExpress.buildRouter(adminJs)
app.use(adminJs.options.rootPath, router)

// Running the server
const run = async () => {
  await mongoose.connect('mongodb://localhost:27017/test', { useNewUrlParser: true })
  await app.listen(8080, () => console.log(`Example app listening on port 8080!`))
}

run()
```

Now (assuming you have MongoDB running) run the server to see if everything is working correctly:

```
node index.js
```

You should see:

```
AdminJS: bundle ready
Example app listening on port 8080!
```

so open: http://localhost:8080/admin and play around with it.

### Hash the password

We see that the password is not hashed. So let’s change that!

We will use [bcrypt](https://www.npmjs.com/package/bcrypt) library for hashing passwords:

```
yarn add bcrypt
```

Now we have to intercept the new action and change the _password_ to encrypted one.

So let’s prepare our model first by renaming the password field in the database to _encryptedPassword_:

```typescript
// Resources definitions
const User = mongoose.model('User', {
  email: { type: String, required: true },
  encryptedPassword: { type: String, required: true },
  role: { type: String, enum: ['admin', 'restricted'], required: true },
})
```

Next, add some options to the AdminJS _User_ model. We will create a new virtual property called _password_ (because we don’t have a password in the database anymore). And we will show it only on an edit page.

Secondly, we create a before action hook which will hash the password.

This is how all of this will look:

```typescript
// Pass all configuration settings to AdminJS
const adminJs = new AdminJS({
  resources: [{
    resource: User,
    options: {
      properties: {
        encryptedPassword: {
          isVisible: false,
        },
        password: {
          type: 'string',
          isVisible: {
            list: false, edit: true, filter: false, show: false,
          },
        },
      },
      actions: {
        new: {
          before: async (request) => {
            if(request.payload.password) {
              request.payload = {
                ...request.payload,
                encryptedPassword: await bcrypt.hash(request.payload.password, 10),
                password: undefined,
              }
            }
            return request
          },
        }
      }
    }
  }],
  rootPath: '/admin',
})
```

As you probably noticed we also hidden _encryptedPassword_ entirely from the UI.

Ok, now is the time to create some real users! So open the Admin Panel and create 2 users: one with the _admin_ role and the other with the _restricted_ role.

Remember their passwords because in the next paragraph we will add a login page.

### Adding login page

[module:@adminjs/express](../installation/plugins/express.md) plugin, which we use for attaching admin to express framework, has the option to authenticate AdminJS users. In order to use it, we have to change the _buildRouter_ function to the _buildAuthenticatedRouter_. Now we can pass the authentication method which will verify an email and a password.

```typescript
// Build and use a router which will handle all AdminJS routes
const router = AdminJSExpress.buildAuthenticatedRouter(adminJs, {
  authenticate: async (email, password) => {
    const user = await User.findOne({ email })
    if (user) {
      const matched = await bcrypt.compare(password, user.encryptedPassword)
      if (matched) {
        return user
      }
    }
    return false
  },
  cookiePassword: 'some-secret-password-used-to-secure-cookie',
})
```

Finally, install the authenticated router dependencies:

```
yarn add cookie-parser express-session
```

and we can run the server again and log in to the app.

### Restricting access to an entire resource

Having all of that, we can now implement a real **Role-Based Access Control**.

To demonstrate this let’s create another collection —  _Cars_ with _name_, _owner_ and _colour_ fields.

```typescript
// Cars collection
const Cars = mongoose.model('Car', {
  name: String,
  color: { type: String, enum: ['black'], required: true }, // Henry Ford
  ownerId: {
    type: mongoose.Types.ObjectId,
    ref: 'User',
  }
})
```

Now, let’s disable modifying users collection so restricted admins will be able to only see the records.

In order to do that we have to add the _isAccessible_ action parameter in User options for: _edit_, _new_ and _delete_ actions:

```typescript
isAccessible: ({ currentAdmin }) => currentAdmin && currentAdmin.role === 'admin',
```

where the _currentAdmin_ is the object we returned in _buildAuthenticatedRouter_ authenticate function.

Now only admins will be able to add new users.

### Restricting access to selected records

_isAccessible_ takes an entire [ActionContext](https://adminjs.page.link/action-interface-code) as an argument. That gives us lots of options. For instance, we can disable editing of cars which does not belong to _restricted_ users (_admins_ will still be able to edit everything).

In order to achieve that we can pass the following function to both _edit_ and _delete_ actions in _Cars_ collection options:

```typescript
isAccessible: ({ currentAdmin, record }) => {
  return currentAdmin && (
    currentAdmin.role === 'admin'
    || currentAdmin._id === record.param('ownerId')
  )
}
```

Seems fine but what with the _new_ action. We have to limit _restricted_ admins to only add their cars. We can do this by filling out _ownerId_ field automatically based on the currently logged-in user.

First, we have to disable this field in the UI:

```json
properties: {
  ownerId: {
    isVisible: {
      edit: false,
      show: true,
      list: true,
      filter: true
    }
  }
}
```

and add a before hook to new action:

```typescript
before: async (request, { currentAdmin }) => {
  request.payload = {
    ...request.payload,
    ownerId: currentAdmin._id,
  }  
  return request
}
```

And that’s it. We’ve just built an Admin Panel with Role-Based Access Control.

And this is the entire code of the application:

```typescript
// Requirements
const mongoose = require('mongoose')
const express = require('express')
const AdminJS = require('adminjs')
const AdminJSExpress = require('@adminjs/express')
const bcrypt = require('bcrypt')

// We have to tell AdminJS that we will manage mongoose resources with it
AdminJS.registerAdapter(require('@adminjs/mongoose'))

// express server definition
const app = express()

// Resources definitions
const User = mongoose.model('User', {
  email: { type: String, required: true },
  encryptedPassword: { type: String, required: true },
  role: { type: String, enum: ['admin', 'restricted'], required: true },
})

// Cars collection
const Cars = mongoose.model('Car', {
  name: String,
  color: { type: String, enum: ['black'], required: true }, // Henry Ford
  ownerId: {
    type: mongoose.Types.ObjectId,
    ref: 'User',
  }
})

// RBAC functions
const canEditCars = ({ currentAdmin, record }) => {
  return currentAdmin && (
    currentAdmin.role === 'admin'
    || currentAdmin._id === record.param('ownerId')
  )
}
const canModifyUsers = ({ currentAdmin }) => currentAdmin && currentAdmin.role === 'admin'

// Pass all configuration settings to AdminJS
const adminJs = new AdminJS({
  resources: [{
    resource: Cars,
    options: {
      properties: {
        ownerId: { isVisible: { edit: false, show: true, list: true, filter: true } }
      },
      actions: {
        edit: { isAccessible: canEditCars },
        delete: { isAccessible: canEditCars },
        new: {
          before: async (request, { currentAdmin }) => {
            request.payload = {
              ...request.payload,
              ownerId: currentAdmin._id,
            }
            return request
          },
        }
      }
    }
  },
  {
    resource: User,
    options: {
      properties: {
        encryptedPassword: { isVisible: false },
        password: {
          type: 'string',
          isVisible: {
            list: false, edit: true, filter: false, show: false,
          },
        },
      },
      actions: {
        new: {
          before: async (request) => {
            if(request.payload.password) {
              request.payload = {
                ...request.payload,
                encryptedPassword: await bcrypt.hash(request.payload.password, 10),
                password: undefined,
              }
            }
            return request
          },
        },
        edit: { isAccessible: canModifyUsers },
        delete: { isAccessible: canModifyUsers },
        new: { isAccessible: canModifyUsers },
      }
    }
  }],
  rootPath: '/admin',
})

// Build and use a router which will handle all AdminJS routes
const router = AdminJSExpress.buildAuthenticatedRouter(adminJs, {
  authenticate: async (email, password) => {
    const user = await User.findOne({ email })
    if (user) {
      const matched = await bcrypt.compare(password, user.encryptedPassword)
      if (matched) {
        return user
      }
    }
    return false
  },
  cookiePassword: 'some-secret-password-used-to-secure-cookie',
})

app.use(adminJs.options.rootPath, router)

// Running the server
const run = async () => {
  await mongoose.connect('mongodb://localhost:27017/test', { useNewUrlParser: true })
  await app.listen(8080, () => console.log(`Example app listening on port 8080!`))
}

run()
```
