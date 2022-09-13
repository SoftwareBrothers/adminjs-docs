# Getting started

The "Getting started" section is an overview and quick explanation of AdminJS ecosystem. Detailed instructions for setting up your AdminJS application and connecting it to your database(s) are described in separate guides (scroll down to [Setup](getting-started.md#undefined) for quick links).

### Overview

An AdminJS application consists of:

* a core package
* a plugin (for a framework of your choice)
* an adapter for (for a ORM/ODM of your choice)

### Packages

First of all, install the core package:

```bash
$ yarn add adminjs
```

Next, install one of our plugins:

```bash
$ yarn add @adminjs/express                # for Express server
$ yarn add @adminjs/nestjs                 # for Nest server
$ yarn add @adminjs/hapi                   # for Hapi server
$ yarn add @adminjs/koa                    # for Koa server
$ yarn add @adminjs/fastify                # for Fastify server
```

Finally, install an adapter:

```bash
$ yarn add @adminjs/typeorm                # for TypeORM
$ yarn add @adminjs/sequelize              # for Sequelize
$ yarn add @adminjs/mongoose               # for Mongoose
$ yarn add @adminjs/prisma                 # for Prisma
$ yarn add @adminjs/mikroorm               # for MikroORM
$ yarn add @adminjs/objection              # for Objection
```

### Setup

After you have installed AdminJS dependencies, proceed to:

* [Plugins](plugins/) section for instructions on how to setup AdminJS with your framework.
* [Adapters](adapters/) section for instructions on how to connect your AdminJS instance to your database.
