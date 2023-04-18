---
description: >-
  For physical applications of AdminJS a pdf generator can come in handy, either
  for generating shipping labels or work orders.
---

# PDF Generator

In order to provide PDF generating capabilities within your AdminJS instance you will have to take advantage of the custom components feature and the [jsPDF library](https://github.com/parallax/jsPDF).

First, create a new instance of the Component Loader, and create a Components object that will hold the custom components.

{% code title="index.ts" %}
```typescript
const componentLoader = new ComponentLoader()

const Components = {
    PDFGenerator: componentLoader.add('GeneratePDF', './pdfgenerator.component')
}
```
{% endcode %}

{% hint style="warning" %}
Remember to [pass the Component Loader instance into your AdminJS config](https://docs.adminjs.co/ui-customization/writing-your-own-components)!
{% endhint %}

Then, we will have to append the PDF generator to a resource and add a handler for the record passed within context.

{% code title="order.resource.ts" %}
```typescript
const orderResource = {
    resource: Order,
    options: {
        actions: {
            PDFGenerator: {
                actionType: 'record',
                icon: 'GeneratePdf',
                component: Components.PDFGenerator,
                handler: (request, response, context) => {
                    const { record, currentAdmin } = context
                    return {
                        record: record.toJSON(currentAdmin),
                        url: pdfgenerator(record.toJSON(currentAdmin))
                    }
                }
            }
        }
    }
}
```
{% endcode %}

With that ready, we can focus on the PDF generator function.

{% code title="pdfgenerator.ts" %}
```typescript
import { RecordJSON } from 'adminjs'
import { jsPDF } from 'jspdf'

const pdfGenerator = (record: RecordJSON): string => {
  const { params } = record
  const doc = new jsPDF()
  
  doc.text(params.orderNum, 10, 10) // example database column called orderNum
  doc.text(params.shippingAddress, 150, 10) // example database column called shippingAddress
  
  const filename = `/${params.id}.pdf`
  doc.save(`./pdfs${filename}`)
  
  return filename
}

export default pdfGenerator
```
{% endcode %}

Now that our PDFs can be created from the record passed within context, we will need to create a place where the pdfs will be stored. Create a folder called 'pdfs', which we will make public through express.

```
.
├── index.ts
├── order.resource.ts
├── pdfHandler.ts
├── pdfgenerator.component.tsx
├── pdfgenerator.ts
└── pdfs
    └── ...
```

{% hint style="warning" %}
Make sure you set the static path before building the router!
{% endhint %}

{% code title="index.ts" %}
```typescript
import path from 'path'
import * as url from 'url'
// other imports

const __dirname = url.fileURLToPath(new URL('.', import.meta.url))

// ...

app.use(express.static(path.join(__dirname, 'pdfs/')))
```
{% endcode %}

Last, but not least, we need a way to open the freshly generated PDF file. Since we've already defined the custom component we're going to stick to the same naming scheme.

{% code title="pdfgenerator.component.tsx" %}
```tsx
import React, { useEffect } from 'react'
import { ApiClient, ActionProps } from 'adminjs'
import { Loader } from '@adminjs/design-system'

const GeneratePdf: React.FC<ActionProps> = (props) => {
  const { record, resource } = props
  const api = new ApiClient()

  useEffect(() => {
    api.recordAction({
      recordId: record.id,
      resourceId: resource.id,
      actionName: 'PDFGenerator'
    }).then((response) => {
      window.location.href = response.data.url
    }).catch((err) => {
      console.error(err)
    })
  }, [])

  return <Loader />
}

export default GeneratePdf
```
{% endcode %}

