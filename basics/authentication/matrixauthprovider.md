---
description: Matrix User Authentication for the @adminjs/matrix plugin.
---

# MatrixAuthProvider

## Authentication - Matrix User Authentication

[➡️ Plugin - @adminjs/matrix](../../installation/plugins/matrix.md)

***

### Matrix User Authentication

Authenticate users directly against your Matrix server using the `MatrixAuthProvider`.

#### Setup Example

```typescript
import { MatrixAuthProvider } from '@adminjs/matrix';
import componentLoader from './component-loader.js';

const provider = new MatrixAuthProvider({
  baseUrl: process.env.MATRIX_BASE_URL,
  componentLoader,
});

export default provider;
```

#### Notes

* Users will be authenticated directly with their Matrix username and password.
* This method does **not** use a shared token – authentication is user-specific.
* You must configure the `MATRIX_BASE_URL` environment variable correctly for your Matrix server.

***

### Related

* [Plugin Setup - @adminjs/matrix](../../installation/plugins/matrix.md)
