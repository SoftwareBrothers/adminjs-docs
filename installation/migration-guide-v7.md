# Migration Guide v7

AdminJS v7 comes with a lot of new features but also many potential breaking changes. If you're a developer that started working on an AdminJS project prior to the version 7 release, this guide should help you with the migration.

## Compatibility List

Below you can find a list of AdminJS packages that are compatible with version 7 changes.

* `adminjs` (7.x.x)
* `@adminjs/design-system` (4.x.x)
* `@adminjs/express` (6.x.x)
* `@adminjs/fastify` (4.x.x)
* `@adminjs/hapi` (7.x.x)
* `@adminjs/koa` (4.x.x)
* `@adminjs/nestjs` (6.x.x)
* `@adminjs/mikroorm` (3.x.x)
* `@adminjs/mongoose` (4.x.x)
* `@adminjs/objection` (2.x.x)
* `@adminjs/prisma` (4.x.x)
* `@adminjs/sequelize` (4.x.x)
* `@adminjs/sql` (2.x.x)
* `@adminjs/typeorm` (5.x.x)
* `@adminjs/passwords` (4.x.x)
* `@adminjs/logger` (5.x.x)
* `@adminjs/import-export` (3.x.x)
* `@adminjs/upload` (4.x.x)
* **NEW** `@adminjs/themes` (1.x.x)

## ESM Support

With version 7 AdminJS has fully moved to ESM and will no longer support CJS projects.

We do not provide any specific migration tutorial for migrating to ESM, we suggest just searching for one - as long as your application is migrated to ESM, AdminJS should work without issues.

{% embed url="https://www.google.com/search?q=nodejs+migrate+to+esm" %}

For Typescript developers, this migration might be easier since the amount of changes you have to make is much less when compared to vanilla CommonJS apps.

## styled-components

AdminJS's design system is built with `styled-components` library which is still incompatible with ESM in its latest official release (`5.3.9`). Version 6 is still in beta - we attempted to use it and while it did work with ESM, it also did bring some additional issues. We've also attempted to use `@emotion/styled` library instead but it is also currently incompatible with ESM.

Eventually, we decided to stick to `styled-components` version `5.3.9`. Our solution was to import the library, make necessary modifications and re-export it.

What this entails, is the modifications that you must make to your custom components:

1. Do not use default import for `styled`. Use named import instead.
2. Import from `@adminjs/design-system/styled-components` instead of `styled-components`

#### Before

```typescript
import styled, { css } from 'styled-components'
import { Box } from '@adminjs/design-system'

const someCss = css`
  /* some css */
`

const StyledBox = styled(Box)`
  ${someCss}
  
  background: black;
`

// ...
```

#### After

```typescript
import { styled, css } from '@adminjs/design-system/styled-components'
import { Box } from '@adminjs/design-system'

const someCss = css`
  /* some css */
`

const StyledBox = styled(Box)`
  ${someCss}
  
  background: black;
`

// ...
```

Please note that `styled-components` are exported from a separate namespace inside `@adminjs/design-system` to avoid mixing contexts.

{% hint style="info" %}
Typescript developers might encounter issues with TS informing you that named exports (such as `createGlobalStyle`) cannot be found in `@adminjs/design-system/styled-components`. These exports are actually present and we could not pinpoint the exact issue which is causing that error to appear.

As a workaround, you can follow the steps below to resolve the problem.

Install `@types/styled-components` as a `devDependency` in your project:

`$ yarn add -D @types/styled-components`

Create a custom `d.ts` file which extends `@adminjs/design-system/styled-components` - for example `./vendor-types/adminjs-styled-components.d.ts`:

<pre class="language-typescript"><code class="lang-typescript"><strong>declare module '@adminjs/design-system/styled-components' {
</strong>  export * from 'styled-components';
}
</code></pre>
{% endhint %}

## AdminJS.bundle removed

`AdminJS.bundle` has been deprecated since version 6. In version 7 it has been completely replaced with `ComponentLoader`. If you still haven't migrated to `ComponentLoader` , please see our custom components tutorial:&#x20;

{% content-ref url="../ui-customization/writing-your-own-components.md" %}
[writing-your-own-components.md](../ui-customization/writing-your-own-components.md)
{% endcontent-ref %}

With version 7, we have also updated other `@adminjs/*` libraries. The libraries which bundle their own custom components now require you to pass your `ComponentLoader` instance, an example would be `@adminjs/passwords`

```typescript
import { ComponentLoader } from 'adminjs'
import passwordsFeature from '@adminjs/passwords'

import { User } from './user.entity.js'

// ...

const componentLoader = new ComponentLoader()

const admin = new AdminJS({
  componentLoader,
  resources: [{
    resource: User,
    options: {},
    features: [passwordsFeature({
      componentLoader,
      // the rest of the feature's config
    })]
  }]
})
```

The example above applies to all other features which use `ComponentLoader`.

## Overriding the Login page

Previously in order to override the Login page, you had to create an AdminJS instance and then use `overrideLogin` method:

```typescript
admin.overrideLogin({ component: LoginComponent })
```

In version 7 all components are now overridden in the same way, the login page is no exception:

```typescript
componentLoader.override('Login', <path to your login component>)
```

Prior to version 7, there were some additional drawbacks to overriding the Login page, mostly to it being Server Side Rendered, thus limiting your options for customization. In version 7, the Login page is a Single Page Application that allows you to modify the Login page freely.

## Login page translations

The default Login page now uses different translation keys than before. All its translations have been moved from either `messages`, `properties`, `labels` to its own space in `components` section.

```json
  "components": {
    "Login": {
      "welcomeHeader": "Welcome",
      "welcomeMessage": "to AdminJS - the world's leading open-source auto-generated admin panel for your Node.js application that allows you to manage all your data in one place",
      "properties": {
        "email": "Email",
        "password": "Password"
      },
      "loginButton": "Login"
    }
  },
```

To learn more about localization changes, please navigate to the Internationalization section:

{% content-ref url="../tutorials/internationalization-i18n.md" %}
[internationalization-i18n.md](../tutorials/internationalization-i18n.md)
{% endcontent-ref %}

## Design System

There are a few significant changes related to the design-system.&#x20;

#### General changes

The main color (`primary100`) is set to buttons, links, avatar, navigation, table row selections, and illustrations.

All the changes can be found in our demo application (navigate to `Pages` > `Design System Examples`):

{% embed url="https://adminjs-demo.herokuapp.com/admin" %}
Demo
{% endembed %}

#### Changed values for variant and color props

To simplify theme customization in AdminJS we modified available props for `Button` and `Box` components from `@adminjs/design-system`. These props are: `variant` and `color`

Previously, you could use the following values for `variant`: `primary, secondary, danger` and `success`. These values were relevant to color. Currently, `variant` attribute modifies the appearance of a button: `text, outlined, contained` and `light`.  Previous `variant` values have been moved to the new `color` attribute. `Button` default values  are: `variant="text"` and `color="primary"`.

`Box` variant attributes are: `card`, `container`, `transparent`, `grey` or `white`. The `color` attribute can be assigned similarly to how it's done with `Button`.

#### Before

```jsx
<Button variant="primary"> 
  Click me
</Button>
```

#### After

```jsx
<Button variant="contained" color="primary">
  Click me
</Button>
```

#### Disclaimer

There is a chance that some packages (React Select, Tip Tap) might not include type definitions. This is caused by their lack of support for ESM.

## Localization

The biggest change to the internationalization feature is the removal of translations from the backend and moving it client-side. The backend is currently responsible for returning the payload with the translation codes. We have made a few changes in the `locale` object in the AdminJS config. The configuration has been expanded with a couple of new options:&#x20;

* `availableLanguages`&#x20;
* `localeDetection`&#x20;
* `withBackend`

AdminJS contains core translations in a few languages. The default language is set to `en`.

The locale object can be empty (null), however, if you want to provide the user with the ability to dynamically change the language, you will have to provide the `availableLanguages` key with an array of translations.

```javascript
locale: { 
  language: 'pl', // default language
  availableLanguages: ['en', 'pl'], 
}
```

Users can use the last selected language stored in the browser cache by setting `localeDetection` variable to `true`,

```javascript
locale: { 
  language: 'pl', 
  availableLanguages: ['en', 'pl'], 
  localeDetection: true, 
}
```

You can extend or change the default translations by passing the language `translations` object into `locale` config object. Below is a simple example:

```javascript
locale: { 
  language: 'pl', 
  availableLanguages: ['en', 'pl'], 
  localeDetection: true, 
  translations: { 
    pl: { 
      messages: { 
        welcomeOnBoard_title: 'Nowy tytuł pulpitu', 
      }, 
    }, 
    en: { 
      messages: { 
        welcomeOnBoard_title: 'New dashboard title', 
      }, 
    }, 
  }, 
},
```

#### Translations groups

All the translation keys remain the same except for two new ones:

* **components** - translations for components&#x20;
* **pages** - translations for custom pages

#### Translations in custom components

You can now dynamically translate custom components and pages with  AdminJS config.

See the example based on a simple custom component:

```typescript
// ... 
import { useTranslation } from 'adminjs'
// ...

const CustomComponent = (props) => {
    const { translateComponent } = useTranslation()
    return <div>{translateComponent('CustomComponent.textToTranslate')}</div>
}
```

You have to add translations to the component's namespace in our locale config to make this work.

```javascript
const options = {
  // ...
  locale: {
    translations: {
      en: {
        components: {
          CustomComponent: {
            textToTranslate: 'This is text to translate'
          },
        }
      }
    }
  }
  // ...
}
```

If you would like to create custom messages to be handled by `useNotice` hook, you have to add our component section to the `messages` namespace. (use  above `options` data)

```javascript
import { useNotice } from "adminjs"
// ...
const sendNotice = useNotice()
// ...
sendNotice({
  message: 'Invalid "type" for relation',
  type: 'error',
})
```

If you want to use your own locale config you will have to change the translation object to be in line with the version 7 update. Below you find a simple example of what has to be changed (let's assume that the default language is `pl` and additional is `en`)

#### Before

```typescript
locale: {
  language: 'pl',
  translations: {
    labels: {
      dashboard: 'Strona główna',
    }
  }
}
```

#### After

```typescript
locale: {
  language: 'pl',
  availableLanguages: ['pl', 'en'],
  translations: {
    pl: {
      labels: {
        dashboard: 'Strona główna',
      }
    },
    en: {
      labels: {
        dashboard: 'Main page',
      }
    },
  }
}
```

## Demo Application

Our demo application at [https://demo.adminjs.co/admin/login](https://demo.adminjs.co/admin/login) contains all the changes described above. If you're still struggling with the migration, you can take a look at a pull request which introduces these changes to our demo app: [https://github.com/SoftwareBrothers/adminjs-example-app/pull/68](https://github.com/SoftwareBrothers/adminjs-example-app/pull/68/)
