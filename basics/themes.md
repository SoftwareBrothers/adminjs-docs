---
description: '@adminjs/themes'
---

# Themes

{% hint style="info" %}
Themes are only available in AdminJS version 7 or higher.
{% endhint %}

Themes are a new feature of AdminJS version 7 which introduces new UI customization options for developers.

#### Features

* You can provide a custom configuration ([https://styled-system.com/theme-specification/](https://styled-system.com/theme-specification/)) per theme, allowing you to easily modify the general look of your admin panel.
* You can provide a custom `style.css` file per theme if you need to.
* You can override any AdminJS default component per theme. Please note that if you override the same component yourself, your own component will take precedence and will be used across all your themes.
* You can assign different themes to specific users based on their role or whatever logic you wish to use. This is done by setting `theme` in `currentAdmin` object.

## Installation & Usage

### Installation

```bash
$ yarn add @adminjs/themes
```

### Usage

`@adminjs/themes` repository contains public themes which you can easily import into your application. It also provides a CLI which you can use to easily generate and bundle your themes.

#### Importing public themes

AdminJS version 7 comes with two new options you can use when instantiating it. These are `availableThemes` and `defaultTheme`.

* `defaultTheme` is an identifier of a theme which will be used as a default one if your `currentAdmin` does not have `theme` defined.
* `availableThemes` is a list of themes which are available in your application.

If you choose not to provide `defaultTheme` and `availableThemes`, the admin panel will behave the same as before version 7 where you can modify the admin panel's look using `branding.theme`.

```typescript
import { dark, light, noSidebar } from '@adminjs/themes'
import AdminJS from 'adminjs'

// ...

const admin = new AdminJS({
  defaultTheme: dark.id,
  availableThemes: [dark, light, noSidebar],
})
```

In the example above we import three public themes from `@adminjs/themes`

* `dark` which is a dark theme for AdminJS
* `light` which is a light theme for AdminJS (basically, the default theme)
* `noSidebar` is a theme which uses components overrides to remove the sidebar and instead moves the resources to the top bar.

Please note that by following the example above all users that sign into your admin panel will see the `dark` theme. An example of how you can use different themes for different users will be shown later.

#### Importing custom themes

Every theme that you import from `@adminjs/themes` is actually a configuration object in the following format:

```typescript
type ThemeConfig = {
  id: string,                          # Theme ID, example: "my-custom-theme"
  name: string,                        # Example: "My Custom Theme"
  overrides: Partial<ThemeOverride>;   # "styled-system" theme configuration
  bundlePath?: string;                 # Path to your theme's "theme.bundle.js" file
  stylePath?: string;                  # Path to your theme's "style.css" file
}
```

With this knowledge, you can create your own custom themes. Please take a look at the example below where we define a simple custom theme which changes the primary color of the admin panel to `teal`.

```typescript
import AdminJS from 'adminjs'

// ...

const myCustomTheme = {
  id: 'my-custom-theme',
  name: 'My Custom Theme',
  overrides: {
    colors: {
      primary100: 'teal',
    },
  },
}
/* We're leaving "bundlePath" and "stylePath" undefined
since we don't use a css file nor custom components. */

// ...

const admin = new AdminJS({
  defaultTheme: myCustomTheme.id,
  availableThemes: [myCustomTheme],
})
```

#### Custom themes with components overrides

To learn how to bundle components for your themes, refer to CLI section below. This section in turn covers how you should configure your theme to know where to search for your bundle file.

If your theme has custom components, then you must define `bundlePath` in it's configuration and/or `stylePath` if it also has it's `style.css` file. The example below is an extension of what was shown previously.

```typescript
import path from 'path';
import * as url from 'url';

const __dirname = url.fileURLToPath(new URL('.', import.meta.url));

// ...

const myCustomTheme = {
  id: 'my-custom-theme',
  name: 'My Custom Theme',
  overrides: {
    colors: {
      primary100: 'teal',
    },
  },
  bundlePath: `${path.join(__dirname, `../themes/my-custom-theme`)}/theme.bundle.js`,
  stylePath: `${path.join(__dirname, `../themes/my-custom-theme`)}/style.css`,
}

// *
```

To learn more about how to override components in your themes, take a look at how `no-sidebar` theme is implemented in [@adminjs/themes repository](https://github.com/SoftwareBrothers/adminjs-themes/tree/main/src/themes/no-sidebar/components).

All theme-specific components that are theme's overriden components must be placed in theme's `components` directory. Component's file name must match the name of the core component you are overriding. In `no-sidebar`'s theme case, these components are `Sidebar` and `TopBar`.

A full list of overridable components' names can be found in [the core repository](https://github.com/SoftwareBrothers/adminjs/blob/feat/adminjs-v7/src/frontend/utils/overridable-component.ts). If you wish to, you can even override `Application` component and write the entire UI from the scratch.

### Assigning themes to specific users

How you assign a theme to a user is up to the developer working on AdminJS panel. You can create a `theme` field in your users collection, you can also assign themes dynamically when i. e. authenticating. Below you can find an example of how this can be done:

```typescript
import { UserRepository } from './user.repository.js'

/* "authenticate" is an authentication function, please refer to "Plugins"
section to see how to set up an authenticated admin panel. */
const authenticate = (email, password) => {
  /* An example of email/password authentication */
  const userRepository = new UserRepository()
  const user = await userRepository.findByEmail(email)
  
  if (!user) return null
  
  if (await !user.comparePassword(password)) return null
  
  const currentAdmin = {
    id: user.id,
    email: user.email,
    role: user.role,
  }

  /* Assigning themes based on role */
  if (currentAdmin.role === 'Admin') {
    // "Admin" has "dark" theme
    currentAdmin.theme = 'dark'
  } else {
    // Any other role has "light" theme
    currentAdmin.theme = 'light'
  }
  
  return currentAdmin
}
```

## CLI

`@adminjs/themes` comes with a CLI tool which can help you develop your themes.

### Commands

```bash
$ npx adminjs-themes generate <options>
$ npx adminjs-themes bundle <options>
```

#### #generate

`generate` command creates an empty theme (with no modifications) which is prepared to be imported into your AdminJS instance.

**Example**

&#x20;`npx adminjs-themes generate "My Custom Theme"`

**Arguments**

* Name of your theme which will be used as it's `id` in kebab-case format (`My Custom Theme` becomes `my-custom-theme`). It is also used as `name` by default if `--description` is not provided.

**Options**

* `--description` `[string]` - a parameter which sets `name` in your theme configuration.
* `--output` `[string]` - the output directory where the theme will be generated. Defaults to `./src/themes`

#### #bundle

`bundle` command bundles your theme's custom components into `theme.bundle.js` file. Please note that this file should be commited into your source code.

**Example**

`npx adminjs-themes bundle "my-custom-theme"`

**Arguments**

* The ID of the theme you want to bundle. If you do not provide the ID, all themes in your input directory will be bundled.

**Options**

* `--input` `[string]` - the input directory with your AdminJS themes. Default to `./src/themes`. In most cases it should match `--output` from `generate` command.

## Contributing

If you would like to help develop `@adminjs/themes` library, please visit it's [repository](https://github.com/SoftwareBrothers/adminjs-themes).

If you want to share your theme with others, open a pull request where you commit it into `src/themes` directory. Make sure that your pull request contains all relevant files and that it does not modify other themes. Videos or pictures of your theme are welcome.

Make sure to update the `example` inside of the repository.

## Examples

An example usage of themes can be found in it's [repository](https://github.com/SoftwareBrothers/adminjs-themes/tree/main/example).

Themes are also included in our [demo application](https://demo.adminjs.co/).

You can see the themes by signing in using the following credentials schema:

```
Email: <THEME_ID>@example.com
Password: password
```

Example:

```
Email: dark@example.com
Password: password
```

The source code of the demo application can be found in it's [repository](https://github.com/SoftwareBrothers/adminjs-example-app).
