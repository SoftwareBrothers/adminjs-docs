# Content Management System

By default, AdminJS is equipped with a [TipTap](https://tiptap.dev/) editor, which makes it a perfect tool for a Content Management System

#### AdminJS as a Content Management System

To add TipTap to the AdminJS setup you need to change the type of the property holding your content to the `richtext`.

```typescript
const admin = new AdminJS({
    resources: [
      {
        resource: Posts,
        options: {
          properties: {
            postContent: {
              type: 'richtext',
            }
          }
        }
      }
    ]
  })  
```
