# Writing your own Components

## Adding custom components

AdminJS handles all properties and actions using its own, built-in components. These are the familiar text inputs, checkboxes, paginated lists, edit forms etc. However, you can define a custom component for any single property or action instead. This gives you great control over how the data is being displayed and modified.

In general, you need to create an instance of `ComponentLoader`, add your custom components there, pass it in the AdminJS options object and specify which properties/actions are using your custom components and where.

`./components.ts`:

```typescript
import { ComponentLoader } from 'adminjs'

const componentLoader = new ComponentLoader()

const Components = {
    MyInput: componentLoader.add('MyInput', './my-input'),
    // other custom components
}

export { componentLoader, Components }
```

`./my-input.tsx`

```tsx
import React from 'react'

// just some regular React component
const MyInputComponent = () => <input />

export default MyInputComponent
```

`./some-resource.ts`

```typescript
import { Components } from './components.js'

export const SomeResource = {
  resource: Something, // database model
  options: {
    properties: {
      someText: {
        type: 'string',
        components: {
          edit: Components.MyInput, // this is our custom component
        },
      },
    },
  },
}
```

`./index.ts`

<pre class="language-typescript"><code class="lang-typescript">import { AdminJS } from 'adminjs'
import { componentLoader } from './components.js'
import { SomeResource } from './some-resource.js'

const admin = new AdminJS({
    resources: [SomeResource],
    componentLoader, // the loader needs to be added here
    // other options
})

admin.watch() // this builds your frontend code in development environment

<strong>// rest of the adapter and plugin code
</strong></code></pre>

Components for actions are added in the exact same way, only in the action section of a resource instead of properties. Refer to [#action-with-custom-component](../basics/action.md#action-with-custom-component "mention") and [#creating-custom-properties](../basics/property.md#creating-custom-properties "mention") pages for more information.

### Custom Component Structure

Files added to `ComponentLoader` need to expose the component function as a default export. Both `.jsx` and `.tsx` formats are supported.

**Advanced usage:** The path passed in the second argument to the `ComponentLoader.add()` and `ComponentLoader.override()` functions needs to be relative to the file where it was called. If you want to wrap these calls in another function you might find that AdminJS cannot find the correct files. To fix that, pass a third argument with the name of the caller function, like this:

```typescript
import { ComponentLoader } from 'adminjs'

const loader = new ComponentLoader()

export const bundleFile = (key: string, path: string) => {
    loader.add(key, path, 'bundleFile') // `bundleFile` is the name of this function
}
```

### Dependencies

AdminJS uses these dependencies internally, so they are exposed for your code without the need to require them in the `package.json` file:

* [react](https://reactjs.org/)
* [react-dom](https://reactjs.org/)
* [prop-types](https://github.com/facebook/prop-types)

**State management**

* [redux](https://redux.js.org/)
* [react-redux](https://github.com/reduxjs/react-redux)

**Routing**

* [react-router](https://reacttraining.com/react-router/)
* [react-router-dom](https://reacttraining.com/react-router/)

**Styling**

* [styled-components](https://www.styled-components.com/docs)
* [styled-system](https://www.styled-system.com/)

**Other**

* [axios](https://github.com/axios/axios)
* [flat](https://www.npmjs.com/package/flat)
* [react-feather](https://feathericons.com/)

### Props passed to components

In your property and action components, you can use props passed by their _controlling components_.

Currently we have 2 _controlling components_:

* one for an action: [BaseActionComponent](https://adminjs.page.link/base-action) with [ActionProps](https://github.com/SoftwareBrothers/adminjs/blob/master/src/frontend/components/actions/action.props.ts)
* and one for custom property field: [BasePropertyComponent](https://github.com/SoftwareBrothers/adminjs/blob/master/src/frontend/components/actions/action.props.ts) with [BasePropertyProps](https://adminjs.page.link/base-property-props)

Check out their documentation to see available **props**

> Other, internal components (like Dashboard) have either no or different props, see the source code in each case.

## Overriding internal AdminJS components

`ComponentLoader` also has an `.override()` method that lets you replace components used by AdminJS internally by your own custom components. This is useful in cases where you want to change or add behavior to the entire AdminJS app, for example:

* Adding a custom dashboard
* Changing the app layout
* Overriding entire controls (like replacing boolean checkboxes with toggles)
* Customizing component look and feel when theming is insufficient

Overriding component works exactly the same way as adding custom components, but you need to specify the matching name of the component ([here's the list](https://github.com/SoftwareBrothers/adminjs/blob/master/src/frontend/utils/overridable-component.ts)).

The methods are split into `.add()` and `.override()` as a safety layer, so you don't accidentally override an internal component with a custom component of the same name - the functions will throw an error when used for conflicting components. In short, `.add()` won't let you use internal component names and `.override()` requires an internal component name.

## Other customizations

### Theming

We support [Theme](https://adminjs.page.link/theme) compatible with https://system-ui.com/theme standard.

In order to override default colors, fonts, sizes etc., you can put your values in AdminJSOptions.branding.

#### Using style props

AdminJS components are supercharged with multiple props controlling styles. For instance in order to change color of a module:@adminjs/design-system.Button you can pass _backgroundColor_ (bg) from the module:@adminjs/design-system.Theme like that:

```html
<Button bg="primary60"></Button>
```

For all possible options visit the [Theme](https://docs.adminjs.co/Theme.html) description.

#### Adding custom css to components

If using style props is not enough - you can always pass your custom CSS. So for instance let's assume that you would like to overwrite CSS in a Button component. You can do this like that:

```typescript
import { Button } from '@adminjs/design-system'

const MyButton = styled(Button)`
  background-color: #ccc;
  color: ${({theme}) => theme.colors.grey100};
  ...
`
```

We use [styled-components](https://styled-components.com/) under the hood so make sure to check out their docs.

### Reusing UI Components of AdminJS

AdminJS gives you the ability to reuse its component library:

```typescript
import { Label } from '@adminjs/design-system'

const YourComponent (props) => {(
  <Label>Some styled text<Label>
)}
```

> We divide components internally to 2 groups:
>
> * _application components_ - which requires AdminJS, you can think about them as "smart components"
> * and _design system components_ - they don't require AdminJS and you can use them outside of the AdminJS setup.
>
> That is why sometimes you have to import components from 'adminjs' package and sometimes from '@adminjs/design-system'.

Each of the components is described with the playground option, so make sure to check out all the documentation of all the components.

One of the most versatile component is a [BasePropertyComponent](https://adminjs.page.link/base-property-code). It allows you to render any property. Combined with [useRecord](https://adminjs.page.link/use-record) is a powerful tool for building forms.

### Creating Custom Pages

You can also use custom components as full pages by specifying their name in the `pages` object in AdminJS options:

```typescript
import AdminJS from 'adminjs'
import { Components } from './components.js'

new AdminJS({
    pages: {
        myPage: { // name, will be used to build an URL
            handler: // handler code,
            component: Components.MyPage,
            icon: // page icon name
        }
    }
})
```

### Using other AdminJS frontend classes and objects

AdminJS also exposes following classes:

* [ApiClient](https://adminjs.page.link/api-client)
* [ViewHelpers](https://adminjs.page.link/view-helpers)

You can use them like this:

```typescript
import { ApiClient, ViewHelpers } from 'adminjs'
```
