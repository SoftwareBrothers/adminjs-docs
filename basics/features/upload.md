---
description: '@adminjs/upload'
---

# Upload

The upload feature helps organize your files and keep information about them in database.

There is possibility to use different storage for files:&#x20;

* local filesystem
* AWS S3
* Google Cloud Storage

To install the upload feature run:

<pre class="language-shell"><code class="lang-shell"><strong>$ yarn add @adminjs/upload
</strong></code></pre>

The main concept of the upload feature is that it sends uploaded files to an external source. The database keeps the information about path and folder name where the file was stored.

The feature uses following terms

* `key` is the path of the stored file
* `bucket` is the name of the container

First we have to create in our database table where we store information about our files.

Below is written interface for entity. Feel free to add your own fields to store other data connected with file.

```typescript
interface IFile {
  id: number;
  s3Key: string;
  bucket: string;
  mime: string;
  comment: string | null;
}
```

Next, you should decide, where your files will be stored and prepare resource entry for AdminJS

{% tabs %}
{% tab title="Local Filesystem" %}
In this example your local server should have established `bucket` folder and it should be accessible by web browser (via `baseUrl` path)

<pre class="language-javascript"><code class="lang-javascript"><strong>import * as url from 'url'
</strong>// other imports

const __dirname = url.fileURLToPath(new URL('.', import.meta.url))

app.use(express.static(path.join(__dirname, '../public')));
</code></pre>



```typescript
import uploadFeature from '@adminjs/upload';

import { File } from './models/file.js';
import componentLoader from './component-loader.js';

const localProvider = {
  bucket: 'public/files',
  opts: {
    baseUrl: '/files',
  },
};

export const files = {
  resource: File,
  options: {
    properties: {
      s3Key: {
        type: 'string',
      },
      bucket: {
        type: 'string',
      },
      mime: {
        type: 'string',
      },
      comment: {
        type: 'textarea',
        isSortable: false,
      },
    },
  },
  features: [
    uploadFeature({
      componentLoader, 
      provider: { local: localProvider },
      validation: { mimeTypes: ['image/png', 'application/pdf', 'audio/mpeg'] },
    }),
  ],
};
```
{% endtab %}

{% tab title="AWS S3" %}
```typescript
import uploadFeature from '@adminjs/upload';

import { File } from './models/file.js';
import componentLoader from './component-loader.js';

const AWScredentials = {
  credentials:{
    accessKeyId: 'AWS_ACCESS_KEY_ID',
    secretAccessKey: 'AWS_SECRET_ACCESS_KEY',
  },
  region: 'AWS_REGION',
  bucket: 'AWS_BUCKET',
};

export const files = {
  resource: File,
  options: {
    properties: {
      s3Key: {
        type: 'string',
      },
      bucket: {
        type: 'string',
      },
      mime: {
        type: 'string',
      },
      comment: {
        type: 'textarea',
        isSortable: false,
      },
    },
  },
  features: [
    uploadFeature({
      componentLoader,
      provider: { aws: AWScredentials },
      validation: { mimeTypes: ['application/pdf'] },
    }),
  ],
};
```
{% endtab %}

{% tab title="Google Cloud Storage" %}
<pre class="language-typescript"><code class="lang-typescript">import uploadFeature from '@adminjs/upload';

import { File } from './models/file.js';
import componentLoader from './component-loader.js';

<strong>const GCScredentials = {
</strong>  serviceAccount: 'SERVICE_ACCOUNT',
  bucket: 'GCP_STORAGE_BUCKET',
  expires: 0,
};

export const files = {
  resource: File,
  options: {
    properties: {
      s3Key: {
        type: 'string',
      },
      bucket: {
        type: 'string',
      },
      mime: {
        type: 'string',
      },
      comment: {
        type: 'textarea',
        isSortable: false,
      },
    },
  },
  features: [
    uploadFeature({
      componentLoader,
      provider: { gpc: GCScredentials },
      validation: { mimeTypes: ['image/png'] },
    }),
  ],
};
</code></pre>
{% endtab %}
{% endtabs %}

After that add files resource to AdminJS options config

```typescript
import { files } from './resources/files.js';

const adminJsOptions = {
  resources: [
     //...
     files
  ],
  //...
}
```

If you would like to deal with multiple files with single database entry it will be necessary to modify config files

{% tabs %}
{% tab title="Entity" %}
```typescript
import { BaseEntity, Column, CreateDateColumn, Entity, PrimaryGeneratedColumn, UpdateDateColumn } from 'typeorm';

@Entity({ name: 'files' })
export class File extends BaseEntity {
  @PrimaryGeneratedColumn()
  public id: number;

  @Column({ name: 's3_key', nullable: true, type: 'jsonb' })
  public s3Key: string;

  @Column({ nullable: true, type: 'jsonb' })
  public bucket: string;

  @Column({ nullable: true, type: 'jsonb' })
  public mime: string;

  @Column({ nullable: true, type: 'text' })
  public comment: string;

  @CreateDateColumn({ name: 'created_at' })
  public createdAt: Date;

  @UpdateDateColumn({ name: 'updated_at' })
  public updatedAt: Date;
}

```
{% endtab %}

{% tab title="Resource" %}
```typescript
import uploadFeature from '@adminjs/upload';

import { File } from './models/file.js';

const localProvider = {
  bucket: 'public/files',
  baseUrl: '/files',
};

export const files = {
  resource: File,
  options: {
    properties: {
      s3Key: {
        type: 'string',
        isArray: true,
      },
      bucket: {
        type: 'string',
        isArray: true,
      },
      mime: {
        type: 'string',
        isArray: true,
      },
      comment: {
        type: 'textarea',
        isSortable: false,
      },
    },
  },
  features: [
    uploadFeature({
      provider: { local: localProvider },
      multiple: true,
      validation: { mimeTypes: ['image/png', 'application/pdf', 'audio/mpeg'] },
    }),
  ],
};
```
{% endtab %}
{% endtabs %}

