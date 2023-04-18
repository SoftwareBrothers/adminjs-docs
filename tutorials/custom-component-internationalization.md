# Custom component internationalization

AdminJS can be used in multiple languages as described in this [tutorial](internationalization-i18n.md). Creating an international version of our custom components is also possible.

Assume we have created a custom React component as described [here](../ui-customization/writing-your-own-components.md). We can utilize a generic AdminJS hook called useTranslations.

<pre class="language-javascript"><code class="lang-javascript"><strong>// ... 
</strong>import { useTranslations } from "adminjs"
// ...

const CustomComponent = (props) => {
    const {translateComponent} = useTranslations()
    return &#x3C;div> {translateComponent('CustomComponent.textToTranslate')} &#x3C;/div>
}
</code></pre>

We should also add translations to the component's namespace in our locale config to make this work.

```javascript
const options = {
  // ...
  locale: {
    translations: {
      en: {
        components: {
          CustomComponent: {
            textToTranslate: 'This is text to translate'
          }
        }
      }
    }
  }
  // ...
}
```

If we would like to create custom messages to be handled by useNotice hook we have to add our component section to the messages namespace.

```javascript
const options = {
  // ...
  locale: {
    translations: {
      en: {
        messages: {
          CustomComponent: {
            componentMessage: 'This is a message from a custom component'
          }
        }
      }
    }
  }
  // ...
}
```

Usage:

```javascript
import { useNotice } from "adminjs"
// ...
const sendNotice = useNotice()
// ...
sendNotice({
  message: 'CustomComponent.componentMessage',
  type: 'error',
})
```

