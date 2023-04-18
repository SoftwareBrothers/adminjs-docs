# What's new in v7?

This sections covers all new features that will be available in version 7 of `adminjs` core library and  compatible `@adminjs/*` packages.

Please make sure to read our [Migration Guide v7](migration-guide-v7.md) article to learn which changes are necessary to migrate if you're already using AdminJS.

## AdminJS is now only ESM-only compatible

With version 7 AdminJS has fully moved to ESM and will no longer support CJS projects.

While we have mixed feelings about the actual benefits ESM brings, it's considered to be a future standard. A migration from CJS to ESM itself is a breaking change. Because of this, a major release seemed to be the correct place for this transition.

However, with this transition, we are also abandoning the support for CJS. The reason for this is that dual packaging of CJS/ESM is a complex topic that we'd decided not to delve deeper into. We've had many issues with libraries that support both at the same time during our migration work, some of them are still not completely resolved and we had to use some temporary (hopefully) workarounds.

## The login page is now SPA

Previously, the login page had been server side rendered which limited developers in how they can customize it without overriding the server's endpoints. An example would be that you couldn't use React hooks or custom events in your login page component, being able to only modify the general look of the HTML form.

This has been changed in version 7 of AdminJS by making the login page a SPA plus it can now be overriden similarly to other custom components.

## Design System updates

A lot of design system components have received a minor visual refresh in order to make the default user interface cleaner.

We have also introduced two new design system components which you can use in your projects:

* `Avatar`
* `Tabs`

### Avatar

`Avatar` is a new component which displays a rounded image of your choice and other content (for example the first letter of your name) in the middle of it.

```typescript
import { Avatar } from "@adminjs/design-system"

export const ExampleAvatar = (props) => {
  const { avatarUrl, email } = props

  return (
    <Box>
      <Avatar src={avatarUrl} alt={email}>
        {email.charAt(0)}
      </Avatar>
    </Box>
  )
}
```

### Tabs

`Tabs` is a new component which allows you to group your content into separate tabs.

```typescript
import { Tab, Tabs, Box } from '@adminjs/design-system'
import React, { useState } from 'react'

export const ExampleTabs = () => {
  const [selectedTab, setSelectedTab] = useState('first')

  return (
    <Tabs currentTab={selectedTab} onChange={setSelectedTab}>
      <Tab id="first" label="First tab">
        First
      </Tab>
      <Tab id="second" label="Second tab">
        <Box color="primary100">Second</Box>
      </Tab>
      <Tab id="third" label="Third tab">
        Third
      </Tab>
    </Tabs>
  )
}
```

`@adminjs/design-system` version 4 comes with some breaking changes. Make sure you read our [migration guide](migration-guide-v7.md) to help you get past them.

Additionally, you can find new and updated design system components in our [Storybook](https://storybook.adminjs.co/).

## Themes

`@adminjs/themes` introduces new UI customization options for developers.

#### Features

* You can provide a custom configuration ([https://styled-system.com/theme-specification/](https://styled-system.com/theme-specification/)) per theme, allowing you to easily modify the general look of your admin panel.
* You can provide a custom `style.css` file per theme if you need to.
* You can override any AdminJS default component per theme. Please note that if you override the same component yourself, your own component will take precedence and will be used across all your themes.
* You can assign different themes to specific users based on their role or whatever logic you wish to use. This is done by setting `theme` in `currentAdmin` object.

## Translations are now client-side only

The localization is now entirely client-side. Please read our [migration guide](migration-guide-v7.md) to learn how to adjust your existing translations.

## Language Selector

By providing `availableLanguages` in your `locale` configuration, you now give a choice for users to select a language they want to see the admin panel in.

```
locale: { 
  language: 'pl', // default language
  availableLanguages: ['en', 'pl'], 
}
```

## You can now translate pages in sidebar

Until now, the name of a page had been shared between what's visible in sidebar and what's present in your browser's address bar. In version 7 you are now able to translate your custom page's label in the sidebar.

```typescript
const admin = new AdminJS({
  pages: {
    myCustomPage: { /* */ },
  }
})
```

The example above will result in the page name appearing as `pages/myCustomPage` in the browser's address pathname, but the sidebar will display it as `My Custom Page`. If you'd like to change its label in the sidebar or define a different name per language, you can now do it in your `locale` settings.

```typescript
"pages": {
  "myCustomPage": "Some Custom Page"
}
```

`pages` is a new section you can define in your translations configuration object.

## There is a new section for translating components

Similarly to `pages`, there's a new `components` section in locale configuration which should help you group your translations by scoping them to your custom components. Additionally, this is what will be used from now on to translate messages in default components from `@adminjs/design-system`.

```
components: { 
  DropZone: {
    placeholder: "Спуштете ја вашата датотека овде или кликнете за да пребарувате",
    acceptedSize: "Максимална големина: {{maxSize}}",
    acceptedType: "Поддржува: {{mimeTypes}}",
    unsupportedSize: "Датотека {{fileName}} е преголем",
    unsupportedType: "Датотека {{fileName}} има неподдржан тип: {{fileType}}"
  }
},
```

The example above shows how you can change the messages in `DropZone` component per language. Previously, the messages had been hardcoded in English.

```typescript
import { DropZone } from '@adminjs/design-system'
import { useTranslation } from 'adminjs'

export const ExampleDropZone = () => {
  const { translateComponent } = useTranslation()
  
  return (
     <Box>
       <Label>Attachment</Label>
       <DropZone
         validate={{ maxSize: 102400, mimeTypes: ['application/pdf'] }}
         translations={translateComponent('DropZone', { returnObjects: true })}
       />  
    </Box>
  )
}
```

## Customized notifications

With version 4 of the design-system the `MessageBox` component has been improved by adding more color variants like `info`, `danger`, `success` and `warning`.  You can also put children in this component to see the extra paragraph below the title.

By default notifications from the hook `useNotice` are set to the `info` variant and they show translated messages from `messages` in locale by code. The example below shows how interpolation can work with a simple notice:

```typescript
// locale
{
  messages: {
    someErrorKey: "Message with interpolation {{ someParams }}" 
  }
}

// component
const addNotice = useNotice();

addNotice({
  message: 'someErrorKey',
  options: {
    someParams: ['param1', 'param2'].join(', '),
  },
})
```

As you can see you can put i18n options in `options` the param to notice. Please notice that you should use the message's keys from the locale but you can also use the message as a fallback.

We have extended the `AppError` instance with notification options to allow the backend to manipulate translation options or notification variants.

```typescript
throw new AppError('someErrorKey', { someData: 'some data for UI' }, { options: { someParams: ['param 1', 'param 2'].join(', ') }})
```

## Demo

You can find all the changes described above in our [demo application](https://demo.adminjs.co).
