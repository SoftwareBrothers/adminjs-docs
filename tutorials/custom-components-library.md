---
description: '@adminjs/custom-components'
---

# Custom components library

If standard library components are not enough there is available library with custom ones. It is  growing continuously to fulfill various users needs.

```shell
$ yarn add @adminjs/custom-components
```

Usage is similar to using [own components](broken-reference) instead writing own component just import one from library.

`./components.ts`

```typescript
import { ComponentLoader } from 'adminjs'
import bundle from '@adminjs/custom-components'

const componentLoader = new ComponentLoader()

const Components = {
  // other custom components
  // 'CustomComponent' is the component name from library
  CustomComponent: bundle(componentLoader, 'CustomComponent'),
}

export { componentLoader, Components }
```
