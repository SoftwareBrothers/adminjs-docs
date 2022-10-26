---
description: '@adminjs/import-export'
---

# Import & Export

AdminJS offers a feature called "@adminjs/import-export" which solves this use case. You can utilize `csv` , `json` or `xml` files

To install add the package to your project.

```shell
$ yarn add @adminjs/import-export
```

Then add one line (`features`) to resource entry in AdminJS config.

```javascript
import importExportFeature from '@adminjs/import-export';

...

{
  resource: Entity,
  features: [
    importExportFeature(),
  ],
}
```

After that you should see two buttons (Import, Export) in the right top corner of  the resource view)

Please remember that field names in imported files should be the same as in database model.
