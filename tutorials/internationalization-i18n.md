# Internationalization (i18n)

{% hint style="warning" %}
This article is outdated and it may contain information that is no longer up-to-date. It will be re-written soon!
{% endhint %}

AdminJS has the default set of texts prepared in English language. But nothing stands in a way for you to change each of them or even translate AdminJS to a different language.

### Locale option and basic translations

All the translations can be overridden by using [AdminJSOptions#locale](https://adminjs.page.link/options-interface) property.

So in order to define how a `new` action is named, simply override it's translation:

```typescript
const options = {
  locale: {
    translations: {
      actions: {
        new: 'Let\'s create',
      }
    }
  }
}

const adminJs = new AdminJS(options)
...
```

but also you can override the name of a `new` action only for a specific resource:

```typescript
const options = {
  locale: {
    translations: {
      actions: {
        new: 'New',
      },
      resources: {
        Article: {
          actions: {
            new: 'New Article'
          }
        }
      }
    }
  }
}
```

### Namespaces

All the translation keys are divided into the following groups:

* **actions** - translations for all [actions](https://adminjs.page.link/action-interface-code) - both default actions, and those created by you.
* **buttons** - translations for all kinds of buttons.
* **messages** - translations for all messages in the app
* **labels** - translations for all labels - usually one word. Labels are used to translate resource names.
* **properties** - translations for all properties.

All of them can be specified globally, or for a specific resource.

### More detailed example

Let's assume that you want to translate your admin panel to polish.

This is how it could look like:

> Take a closer look at this example because it contains a different edge cases like translating the `add new item` button for a particular property, or translating labels for your database enums.

```typescript
const options = {
  locale: {
    language: 'pl',
    translations: {
      actions: {
        new: 'Stwórz nowy',
        edit: 'Edytuj',
        show: 'Detale',
        ...,
      },
      buttons: {
        save: 'zapisz',
        // We use i18next with its pluralization logic.
        confirmRemovalMany_1: 'Potwierdź usunięcie {{count}} rekordu',
        confirmRemovalMany_2: 'Potwierdź usunięcie {{count}} rekordów',
        ...
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
        'isAdmin.false': 'normalny'
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
            name: 'Tytuł'
          }
        }
      }
    }
  }
}
```

### Using i18n in your application and in AdminJS

In the case when you use i18next in your app already, you need to initialize AdminJS in your i18next init callback. In this way AdminJS will add new translations to existing ones:

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

[Action#before](https://adminjs.page.link/action-interface-code) and [Action#after](https://adminjs.page.link/action-interface-code) hooks come with an Action#ActionContext param. It combines all the [TranslateFunctions](https://adminjs.page.link/translate-funtions) like `translateButton`, `translateLabel` etc.

so you can use them like this:

```typescript
// before Hook
{
  after: async (response, request, context) => {
    const { translateMessage } = context
    ...
  }
}
```

If you want to use translations in your components - you can use [useTranslation](https://adminjs.page.link/use-translation) hook.

### More options...

On the backend, we use [https://www.i18next.com/](https://www.i18next.com/) library. So make sure to check out their docs to read more about all the possible options.

Also, you can always check the default English translation file available [in our repo here](https://github.com/SoftwareBrothers/adminjs/blob/v2.0/src/locale/en.ts).
