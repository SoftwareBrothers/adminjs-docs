# Deploying the application using CLI

AdminJS Cloud Hosting comes with a CLI tool which you can use locally or inside your CI/CD to deploy your application.

## Installation

{% code overflow="wrap" %}
```bash
$ npm i -g @adminjs/cloud-cli
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

## Commands

As of version `1.1.0` the CLI only allows you to deploy your application.

To use `@adminjs/cloud-cli` you must first request an application in [Pricing](https://adminjs.co/pricing) page and generate an API Key & API Secret.

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
$ adminjs-cloud deploy --apiKey=<string> --apiSecret=<string> --config=[string]
```
