# Serverless

AdminJS can be deployed on serverless platforms such as [Vercel](https://vercel.com/) or AWS Lambda.
However, special attention is needed when it comes to bundling custom components and serving static assets.

This guide will walk you through:

1. **Bundling custom AdminJS components for serverless deployment**
2. **Serving AdminJS static files in a serverless environment**

---

{% hint style="info" %}
Example repository with AdminJS v6 & Vercel Serverless:\
[https://github.com/puleugo/nestjs-adminjs-serverless](https://github.com/puleugo/nestjs-adminjs-serverless)
{% endhint %}


## Bundling AdminJS Components

When deploying AdminJS, any custom React components you add need to be bundled into static files that the client can access.
AdminJS does not provide an official CLI for this process, but the [`@adminjs/bundler`](https://github.com/SoftwareBrothers/adminjs-bundler) package is recommended.

### Choosing the Right Bundler Version

* If your `tsconfig.json > compilerOptions.module` is **commonjs**, use `@adminjs/bundler@^2.0.0` or before.
* If you are using **ESM** (ECMAScript Modules), use `@adminjs/bundler@^3.0.0` or after.

{% hint style="info" %}
Check your module system before installing the bundler to avoid compatibility issues.
{% endhint %}


#### Example: Using CommonJS ([bundler@2.x](mailto:bundler@2.x))

**Install**

NPM:
```bash
npm install @adminjs/bundler@^2.0.0
```
Yarn:
```bash
yarn add @adminjs/bundler@^2.0.0
```

**Create your component loader:**

```ts
// src/admin/component/index.ts
import { ComponentLoader } from 'adminjs';

export const componentLoader = new ComponentLoader();
export const components = {
  NotEditableInput: componentLoader.add('NotEditableInput', './NotEditableInput'),
};
```

**Create a bundler entry:**

```ts
// src/bundler.ts
import { bundle } from '@adminjs/bundler';
import { join } from 'path';

void (async () => {
  await bundle({
    customComponentsInitializationFilePath: 'src/admin/component/index.ts',
    destinationDir: 'dist/public',
  });
})();
```

**Add to your build script (e.g. in `package.json`):**

```json
{
  "scripts": {
    "build": "nest build && node dist/bundler.ts"
  }
}
```

Running `yarn build` will output bundled assets in `dist/public`.

---

## Serving Bundled Files in Serverless

### Using Vercel (Recommended Example)

Update your `vercel.json` to serve both your server (API) and static assets:

```json
{
  "version": 2,
  "builds": [
    {
      "src": "dist/main.js",
      "use": "@vercel/node",
      "config": {
        "includeFiles": ["dist/**/*"]
      }
    },
    {
      "src": "dist/public/**/*",
      "use": "@vercel/static",
      "config": {
        "outputDirectory": "dist/public"
      }
    }
  ],
  "routes": [
    { "src": "/public/(.*)", "dest": "/dist/public/$1", "methods": ["GET"] },
    { "src": "/(.*)", "dest": "/dist/main.js" }
  ]
}
```

This configuration ensures:

* Your main server is handled by Vercel’s serverless function.
* Your static assets (custom AdminJS components) are served from `/public/`.

---

## Loading Bundled Assets in AdminJS

In your AdminJS module configuration, make sure to set `assetsCDN` to the correct static URL:

```ts
@Module({
  imports: [
    AdminJsModule.createAdminAsync({
      useFactory: () => ({
        adminJsOptions: {
          rootPath: '/admin',
          assetsCDN: 'https://your-serverless-domain.vercel.app/public/', // Must end with /
        },
      }),
    }),
  ],
})
export class AdminModule implements OnModuleInit {
  async onModuleInit() {
    if (process.env.NODE_ENV === 'development') {
      await adminjs.watch();
    }
  }
}
```
