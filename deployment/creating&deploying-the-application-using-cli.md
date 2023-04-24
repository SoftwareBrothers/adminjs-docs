# AdminJS Cloud CLI for creating and deploing application

AdminJS Cloud Hosting comes with a CLI tool which you can use locally or inside your CI/CD to deploy your application.

## Installation

{% code overflow="wrap" %}

```bash
npm i -g @adminjs/cloud-cli
```

{% endcode %}

## Configuration

`@adminjs/cloud-cli` relies on configuration file to be present in your source code. The default file name is `adminjs-cloud.json` but you can provide a custom file path using `--config` option in a CLI command.

### Options

```
include        string[]        A list of files/directories you wish to deploy.
```

### Example

{% code title="adminjs-cloud.json" %}

```json
{
  "include": [
    "public",
    "dist",
    "src",
    ".env",
    "package.json",
    "tsconfig.json",
    "yarn.lock"
  ]
}
```

{% endcode %}

### Starting the application

Currently, AdminJS Cloud Hosting requires `start` script to be present in your `package.json` file. This is the command you use to start your application:

{% code title="package.json" %}

```json
{
  ...,
  "scripts": {
    ...,
    "start": "node app.js"
  }
}
```

{% endcode %}

In the future, we plan to extend application's configuration so that you can provide a custom start command.

## Commands

As of version `1.2.0` the CLI only allows you to create and deploy your application.

To use `@adminjs/cloud-cli` you must first request an application in [Pricing](https://adminjs.co/pricing) page and generate an API Key & API Secret.

### #create

The `create` command allows you to create basic admin JS application with basic authentication. The CLI only generates only code for running it, you have to use commands `yarn && yarn build && yarn start` to check if setup is complete succesfully, if you fallow all the steps correctly.

#### Parameters

```
name          string        required        The name of your application
database      string        required        The connection string todatabase eg. `postgres://adminjs:adminjs@localhost:5432/adminjs`
apiKey        string        required        Your API Key
apiSecret     string        required        Your API Secret
config        string        optional        Path to your configuration file (relative to PWD)
```

#### Usage

```bash
adminjs-cloud create --name=<string> --database=<string> --apiKey=<string> --apiSecret=<string>
```
### #deploy

The `deploy` command allows you to deploy your source code. The CLI assumes your code is already built and whatever files you choose to `include` in your configuration file are enough to start your application.

#### Parameters

```
apiKey        string        required        Your API Key
apiSecret     string        required        Your API Secret
config        string        optional        Path to your configuration file (relative to PWD)
```

#### Usage

```bash
adminjs-cloud deploy --apiKey=<string> --apiSecret=<string> --config=[string]
```
