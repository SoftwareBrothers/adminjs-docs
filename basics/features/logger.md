---
description: '@adminjs/logger'
---

# Logger

AdminJS has some extra plugins which extend its basic functionality. One of them is logger.&#x20;

The logger's purpose is to keep track of selected resources (i.e. tables) Every action performed on resource (new, edit, delete or bulkDelete) is registered in special object and persisted in database. Information about changes are monitored, so we can inspect and compare changes in every field of a table.

Installation is pretty simple. First we have to install `@adminjs/logger` package

```shell
$ yarn add @adminjs/logger
```

We can use any database and any ORM package as described in **Adapters** section.

Then we have to define `Log` entity - the place we will track changes.  Below you can find example

{% tabs %}
{% tab title="Sequelize" %}
```typescript
import { DataTypes, Model } from 'sequelize';

import db from './sequelize.connection.js';
import User from './user.entity.js';

export interface ILog = {
  id: number;
  action: string;
  resource: string;
  userId: number;
  recordId: number;
  recordTitle: string;
  difference: string;
  createdAt: Date;
  updatedAt: Date;
};

export class Log extends Model<ILog> {
  id: number;
  createdAt: Date;
  updatedAt?: Date;
  recordId: number;
  recordTitle: string | null;
  difference: Record<string, unknown> | null;
  action: string;
  resource: string;
  userId: number;
}

Log.init(
  {
    id: {
      type: DataTypes.INTEGER,
      autoIncrement: true,
      primaryKey: true,
    },
    action: {
      type: new DataTypes.STRING(128),
      allowNull: false,
    },
    resource: {
      type: new DataTypes.STRING(128),
      allowNull: false,
    },
    userId: {
      type: DataTypes.INTEGER,
      allowNull: false,
    },
    recordId: {
      type: DataTypes.INTEGER,
      allowNull: false,
    },
    recordTitle: {
      type: new DataTypes.STRING(128),
      allowNull: false,
    },
    difference: {
      type: DataTypes.JSONB,
      allowNull: true,
    },
  },
  {
    sequelize: db,
    tableName: 'logs',
    timestamps: true,
  }
);

export default Log;
```
{% endtab %}

{% tab title="TypeORM" %}
```typescript
import { BaseEntity, Column, Entity, PrimaryGeneratedColumn, CreateDateColumn, UpdateDateColumn } from 'typeorm';

export interface ILog {
  id: number;
  action: string;
  resource: string;
  userId: string | null;
  recordId: number;
  recordTitle: string | null;
  difference: Record<string, unknown> | null;
  createdAt: Date;
  updatedAt?: Date;
}

@Entity({ name: 'logs' })
export class Log extends BaseEntity implements ILog {
  @PrimaryGeneratedColumn()
  public id: number;

  @CreateDateColumn({ name: 'created_at' })
  public createdAt: Date;

  @UpdateDateColumn({ name: 'updated_at' })
  public updatedAt?: Date;

  @Column({ name: 'record_id', type: 'integer', nullable: false })
  public recordId: number;

  @Column({ name: 'record_title', type: 'text', nullable: true, default: '' })
  public recordTitle: string | null;

  @Column({ name: 'difference', type: 'jsonb', nullable: true, default: {} })
  public difference: Record<string, unknown> | null;

  @Column({ name: 'action', type: 'varchar', length: 128, nullable: false })
  public action: string;

  @Column({ name: 'resource', type: 'varchar', length: 128, nullable: false })
  public resource: string;

  @Column({ name: 'user_id', type: 'varchar', nullable: false })
  public userId: string;
}

```
{% endtab %}

{% tab title="MikroORM" %}
```typescript
import { Entity, BaseEntity, PrimaryKey, Property } from '@mikro-orm/core';

export interface ILog {
  id: number;
  action: string;
  resource: string;
  userId: string | null;
  recordId: number;
  recordTitle: string | null;
  difference: Record<string, unknown> | null;
  createdAt: Date;
  updatedAt?: Date;
}

@Entity({ tableName: 'logs' })
export class Log extends BaseEntity<Log, 'id'> implements ILog {
  @PrimaryKey()
  public id: number;

  @Property({ columnType: 'datetime', fieldName: 'created_at', nullable: false })
  public createdAt: Date = new Date();

  @Property({ columnType: 'datetime', fieldName: 'updated_at', nullable: true })
  public updatedAt?: Date = new Date();

  @Property({ columnType: 'integer', fieldName: 'record_id', nullable: false })
  public recordId: number;

  @Property({ columnType: 'varchar', fieldName: 'record_title', nullable: true })
  public recordTitle: string | null;

  @Property({ columnType: 'jsonb', fieldName: 'difference', nullable: true })
  public difference: Record<string, unknown> | null;

  @Property({ columnType: 'varchar', fieldName: 'action', nullable: false })
  public action: string;

  @Property({ columnType: 'varchar', fieldName: 'resource', nullable: false })
  public resource: string;

  @Property({ columnType: 'varchar', fieldName: 'user_id', nullable: false })
  public userId: string;
}

```
{% endtab %}

{% tab title="Mongoose" %}
```typescript
import { model, Schema } from 'mongoose';

export interface Log {
  id: number;
  action: string;
  resource: string;
  userId: string | null;
  recordId: string;
  recordTitle: string | null;
  difference: Record<string, unknown> | null;
  createdAt: Date;
  updatedAt?: Date;
}

export const LogSchema = new Schema<Log>({
  createdAt: { type: Date, default: Date.now },
  updatedAt: { type: Date, default: Date.now },
  recordId: { type: 'String', required: true },
  recordTitle: { type: 'String' },
  difference: 'Object',
  action: { type: 'String' },
  resource: { type: 'String' },
  userId: { type: 'String' },
});

export const LogModel = model<Log>('Log', LogSchema);

```
{% endtab %}

{% tab title="ObjectionJS" %}
```typescript
import { BaseModel } from '../utils/base-model.js';

class Log extends BaseModel {
  id: number;
  recordId: number;
  recordTitle: string;
  difference: Record<string, unknown> | null;
  action: string;
  resource: string;
  userId: string;

  static tableName = 'logs';

  static jsonSchema = {
    type: 'object',
    required: ['recordId', 'action', 'resource', 'userId'],

    properties: {
      id: { type: 'integer' },
      recordId: { type: 'integer' },
      recordTitle: { type: 'string', minLength: 1, maxLength: 128 },
      difference: { type: 'jsonb' },
      action: { type: 'string', minLength: 1, maxLength: 128 },
      resource: { type: 'string', minLength: 1, maxLength: 128 },
      userId: { type: 'string', minLength: 1, maxLength: 128 },
      createdAt: { type: 'string', format: 'date-time', readOnly: true },
      updatedAt: { type: 'string', format: 'date-time', readOnly: true },
    },
  };
}

export default Log;

```
{% endtab %}

{% tab title="Prisma" %}
```
model Log {
  id          Int      @id @default(autoincrement())
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  recordId    Int
  recordTitle String?  @db.VarChar(128)
  difference  Json?    @db.Json
  action      String   @db.VarChar(128)
  resource    String   @db.VarChar(128)
  userId      String   @db.VarChar(128)

}
```
{% endtab %}
{% endtabs %}

Entity`Log` is related to entity `User` because `userId` holds reference to user who made changes&#x20;

There is one thing worth to mention. We have to enable authorization before using this feature, because we will need user's ID for corresponding changes.

To get logger to work, add extra property to resource config. Below you can find simple example of such configuration.

```javascript
import loggerFeature from '@adminjs/logger';

import ResourceModel from './resource.entity.js';
import componentLoader from './component-loader.js';

export default {
  resource: ResourceModel,
  features: [
    loggerFeature({
      componentLoader,
      propertiesMapping: {
        user: 'userId',
      },
      userIdAttribute: 'id',
    }),
  ],
};

```

To have Log resource appear in AdminJS panel, we have to define it first.&#x20;

`@adminjs/logger exports` `createLoggerResource` function which does most of the work for you. You can customize it using it's configuration argument.

```javascript
import { createLoggerResource } from '@adminjs/logger';

import Log from './logs.entity.js';

const config = {
  resource: Log,
  featureOptions: {
    propertiesMapping: {
      recordTitle: 'title' //field to store logged record's title
    ,
    userIdAttribute: 'id', //primary key currently logged user
    resourceOptions: {
      navigation: {
        name: 'SectionName',
        icon: 'iconName'
      }
    }
  }
}

export default createLoggerResource(config)
```

