# Internationalization (i18n)

AdminJS has the default set of texts prepared in some languages. But nothing stands in the way for you to change each of them or even translate AdminJS to a different language.

### Locale option and basic translations

All the translations can be overridden by using [AdminJSOptions#locale](https://adminjs.page.link/options-interface) property.

You can enable translation for all languages built in AdminJS by adding `locale` object to AdminJS options

```typescript
import { locales as AdminJSLocales } from 'adminjs'
// ...
const options = { 
  locale: { 
    language: 'pl', // default language of application (also fallback)
    availableLanguages: Object.keys(AdminJSLocales), 
  }
}
// ...
const adminJs = new AdminJS(options)
```

AdminJS options extend i18n configuration and allow to configure external params like:

| option             | default | description                                                                                                                                          |
| ------------------ | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| language           | en      |  main language of application                                                                                                                        |
| availableLanguages | \['en'] | array of supported langages keys                                                                                                                     |
| localeDetection    | false   | enables locale detections [https://github.com/i18next/i18next-browser-languagedetector](https://github.com/i18next/i18next-browser-languagedetector) |
| withBackend        | false   | enables backend loaded translations [https://github.com/i18next/i18next-http-backend](https://github.com/i18next/i18next-http-backend)               |
| translations       | -       | allows adding or override language translations as `Record<langKey, Locale>`                                                                         |

To set the default language you can use language property

```javascript
locale: { 
  language: 'pl', 
  availableLanguages: ['en', 'pl'], 
}
```

If you would like to constrain available languages to specific you have to modify `availableLanguages` array

```javascript
locale: { 
  language: 'pl', 
  availableLanguages: ['en', 'pl'], 
  localeDetection: true, 
}
```

AdminJS translations can be managed by backend services using [i18next-http-backend](https://github.com/i18next/i18next-http-backend) package. To enable this option set `withBackend` variable to `true`

```javascript
locale: { 
  language: 'pl', 
  availableLanguages: ['en', 'pl'], 
  localeDetection: true, 
}
```

Default translations can be extended or changed by passing `language` translations object into `locale` config object. Below is a simple example:

```javascript
locale: { 
  language: 'pl', 
  availableLanguages: ['en', 'pl'], 
  localeDetection: true, 
  translations: { 
    pl: { 
      messages: { 
        welcomeOnBoard_title: 'Nowy tyluł pulpitu', 
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

### Translations groups

All the translation keys are divided into the following groups:

* **actions** - translations for all [actions](https://adminjs.page.link/action-interface-code) - both default actions, and those created by you.
* **buttons** - translations for all kinds of buttons.
* **messages** - translations for all messages in the app
* **labels** - translations for all labels - usually one word. Labels are used to translate resource names.
* **properties** - translations for all properties.
* **pages** - (new in version 7) translations for your pages labels
* **components** - (new in version 7) a section which allows you to scope translations to components

All of them can be specified globally or for a specific resource.

### More detailed example

Let's assume that you want to translate your admin panel into Polish.

This is what it could look like:

> Take a closer look at this example because it contains a different edge cases like translating the `add new item` button for a particular property, or translating labels for your database enums.

```javascript
locale: {
    language: 'pl',
    availableLanguages: ['en', 'pl'],
    localeDetection: true,
    translations: {
      pl: {
        messages: {
          welcomeOnBoard_title: 'Nowy tyluł pulpitu',
        },
        actions: {
          new: 'Stwórz nowy',
          edit: 'Edytuj',
          show: 'Detale',
        },
        buttons: {
          save: 'zapisz',
          // We use i18next with its pluralization logic.
          confirmRemovalMany_1: 'Potwierdź usunięcie {{count}} rekordu',
          confirmRemovalMany_2: 'Potwierdź usunięcie {{count}} rekordów',
        },
        properties: {
          // labels of properties in all resources with name "name"
          // will be translated to "Nazwa".
          name: 'Nazwa',
          nested: 'Zagniezdzone',
          // this is how nested properties (for nested schemas) can be provided
          'nested.width': 'Szerokość',
          // translate values of boolean property
          'isAdmin.true': 'admin',
          'isAdmin.false': 'normalny',
          // translate values of enums:
          'companySize.small': 'mała',
          'companySize.medium': 'średnia',
          'companySize.big': 'duza',
          // tags is an array and we translate button for this array:
          'tags.addNewItem': 'Dodaj nowy tag',
        },
        labels: {
          // here we translate the name of a resource.
          Comment: 'Komentarze',
        },
        resources: {
          Comment: {
            properties: {
              // this will override the name only for Comment resource.
              name: 'Tytuł',
            },
          },
        },
      },
    },
  },
```

### Using i18n in your application and in AdminJS

In the case that you use i18next in your app already, you need to initialize AdminJS in your i18next init callback. In this way AdminJS will add new translations to existing ones:

```typescript
const loadAdminJS = () => {
  const { adminJs, adminRouter } = admin()
  app.use(adminJs.options.rootPath, adminMiddleware, adminRouter)
  app.use('/admin', adminMiddleware, adminController)
}

i18next.init({...}, (err, t) => {
  loadAdminJS()
})
```

### How to use translations in my custom actions/components

Custom components can also be translated as described (add link) Custom component internationalization

### Overriding language detection

Sometimes it might be useful to override language detection. You can do this by setting lang`u`age value to '`cimode`'.

You can also set `locale` value based on `currentAdmin` object (which holds logged user details)

```javascript
locale: (currentAdmin) => currentAdmin.email === 'specific user email' ? { language: 'cimode' } : locale
```

### More options...

On the backend, we use [https://www.i18next.com/](https://www.i18next.com/) library. So make sure to check out their docs to read more about all the possible options.

Also, you can always check the default English translation file available [in our repo here](https://github.com/SoftwareBrothers/adminjs/blob/v2.0/src/locale/en.ts).
