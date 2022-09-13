---
description: Properties are AdminJS's representation of your model's fields.
---

# Property

## Introduction

Properties are AdminJS's representation of your model's fields. Every adapter used by AdminJS must export a `Property` which is an extension of [BaseProperty](https://adminjs.page.link/base-property-code). In this section you will learn how to override default settings of your resource's properties as well as how to add new properties to your resource using it's configuration object.

## Customizing properties

This tutorial will cover how you can use different [Property options](https://adminjs.page.link/property-options-code) to customize your properties.

### Creating custom properties

Let's assume that you would like to add a new property called `randomPicture` to your `User` resource which would show a randomly generated picture in your `User` resource's `list` and `show` actions.

First, let's create a custom React component which will display the image:

```jsx
import React from 'react'
import { ShowPropertyProps } from 'adminjs'
import { Box } from '@adminjs/design-system'

const RandomPicture: React.FC<ShowPropertyProps> = (props) => {
  // Picsum generates a random 200x200 photo
  const url = 'https://picsum.photos/200'
}

export default RandomPicture
```

```typescript
import AdminJS from 'adminjs'
// other imports

const RANDOM_PICTURE = AdminJS.bundle('./random-picture')

const UserResource = {
  resource: User,
  options: {
    properties: {
      randomPicture: {
        type: 'string',
        components: {
          list: RANDOM_PICTURE,
          show: RANDOM_PICTURE,
        },
      },
    },
  },
}
```

Every property can be further customized. That will be covered in the later parts of this section.

### Displaying a tooltip next to a field in forms

If a field that you are displaying in your form might require additional information to fill in correctly, you may want to use a `description` option in your property's configuration. This option allows you to set a message that will be displayed after hovering over a question mark icon next to a label in your form.

```typescript
const UserResource = {
  resource: User,
  options: {
    properties: {
      links: {
        description: "User's Linkedin/Github/social profiles links",
      },
    },
  },
}
```

You can also use a translation key and define a translation in locale.

```typescript
links: {
  description: "userLinksHint",
},
```

### Defining available values for user to choose from

Although this is not required most of the time because your adapter of choice should be able to load enum values from your models, there can still be a situation where you want a field to be a selection instead of a regular text input. This can be done through `availableValues` option.

```typescript
const UserResource = {
  resource: User,
  options: {
    properties: {
      gender: {
        availableValues: [
          { value: 'male', label: 'Male' },
          { value: 'female', label: 'Female' },
          { value: 'other', label: 'Other' },
          { value: 'notSay', label: 'Rather not say' },
        ],
      },
    },
  },
}
```

### Passing props into HTML element/component

There could be a case where you want to pass extra props to the React component or HTML element which AdminJS is using to render a form field. You can use `props` for this:

```typescript
const UserResource = {
  resource: User,
  options: {
    properties: {
      bio: {
        type: 'textarea',
        props: {
          rows: 20,
        },
      },
    },
  },
}
```

The example above sets [textarea](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/textarea#attr-rows)'s `rows` prop to `20`, the default being `2`.

