# ElasticApmSourceMapUploaderPlugin

This is a port of [ RollbarSourceMapPlugin](https://github.com/thredup/rollbar-sourcemap-webpack-plugin#readme), 
converted to upload to ElasticAPM instead of Rollbar.

The difference between this implementation and [elastic-apm-sourcemap-webpack-plugin](https://github.com/wuct/elastic-apm-sourcemap-webpack-plugin#readme),
is that with [ElasticApmSourceMapUploader](https://github.com/rohan-buchner/elastic-apm-sourcemap-uploader-webpack-plugin#readme), like its base (Rollbar) plugin, the sourcemaps get uploaded during the build pipeline, and not exposed to the outside world.

<hr/>

This is a [Webpack](https://webpack.github.io) plugin that simplifies uploading the sourcemaps,
generated from a webpack build, to [Elastic APM Server](https://www.elastic.co/guide/en/apm/server/7.10/overview.html).

## Prerequisites

**As of version 3.0.0, Webpack 4 is required. This plugin is no longer compatible with Webpack 3 and older.**

## Installation

Install the plugin with npm:

```shell
npm install elastic-apm-sourcemap-uploader-webpack-plugin --save-dev
```

## Basic Usage

An example webpack.config.js:

```javascript
const ElasticApmSourceMapPlugin = require('elastic-apm-sourcemap-uploader-webpack-plugin')
const PUBLIC_PATH = '/';

const webpackConfig = {
  mode: 'production',
  devtool: 'hidden-source-map',
  entry: 'index',
  publicPath: PUBLIC_PATH,
  output: {
    path: 'dist',
    filename: 'index-[hash].js'
  },
  plugins: [new ElasticApmSourceMapPlugin({
    accessToken: '<YOU_ACCESS_TOKEN>',
    serviceName: 'your-service-name',
    version: '4.0.0',
    apmEndpoint: `http://localhost:8200/assets/v1/sourcemaps`,
    publicPath: PUBLIC_PATH,
    silent: false,
    ignoreErrors: false,
  })]
}
```

## Plugin Configuration

You can pass a hash of configuration options to `ElasticApmSourceMapPlugin`.
Allowed values are as follows:

### `accessToken: string` **(required)**

Your elastic apm `api_key`.

### `version: string` **(required)**

A string identifying the version of your code this source map package is for. Typically this will be the full git sha.

### `publicPath: string | function(string): string` **(required)**

The base url for the cdn where your production bundles are hosted or a function that receives the source file local address and returns the url for that file in the cdn where your production bundles are hosted.
You should use the function form when your project has some kind of divergence between url routes and actual folder structure.
For example: NextJs projects can serve bundled files in the following url `http://my.app/_next/123abc123abc123/page/home.js` but have a folder structure like this `APP_ROOT/build/bundles/pages/home.js`.
The function form allows you to transform the final public url in order to conform with your routing needs.

### `includeChunks: string | [string]` **(optional)**

An array of chunks for which sourcemaps should be uploaded.
This should correspond to the names in the webpack config `entry` field.
If there's only one chunk, it can be a string rather than an array.
If not supplied, all sourcemaps emitted by webpack will be uploaded, including those for unnamed chunks.

### `silent: boolean` **(default: `false`)**

If `false`, success and warning messages will be logged to the console for each upload. Note: if you also do not want to see errors, set the `ignoreErrors` option to `true`.

### `ignoreErrors: boolean` **(default: `false`)**

Set to `true` to bypass adding upload errors to the webpack compilation. Do this if you do not want to fail the build when sourcemap uploads fail.
If you do not want to fail the build but you do want to see the failures as warnings, make sure `silent` option is set to `false`.

### `apmEndpoint: string` **(default: `https://localhost:8200/assets/v1/sourcemap`)**

A string defining the Elastic APM API endpoint to upload the sourcemaps to.

### `encodeFilename: boolean` **(default: `false`)**

Set to true to encode the filename. NextJS will reference the encode the URL when referencing the minified script which must match exactly with the minified file URL uploaded to Rollbar.

## Webpack Sourcemap Configuration

The [`output.devtool`](https://webpack.js.org/configuration/devtool/) field in webpack configuration controls how sourcemaps are generated.
The recommended setup for sourcemaps in a production app is to use hidden sourcemaps.
This will include original sources in your sourcemaps, which will be uploaded to Rollbar and NOT to a public location alongside the minified javascript.
The `hidden` prefix will prevent `//# sourceMappingURL=URL_TO_SOURCE_MAP` from being inserted in the minified javascript.
This is important because if the `sourceMappingURL` comment is present,
Rollbar will attempt to download the sourcemap from this url, which negates the whole
purpose of this plugin. And since you are not uploading sourcemaps to a public location,
Rollbar would not be able to download the sourcemaps.

### webpack.config.js

```json
output: {
  devtool: 'hidden-source-map'
}
```

## App Configuration

- The web app should have [RUM.JS](https://www.elastic.co/guide/en/apm/agent/rum-js/5.x/install-the-agent.html) Agent installed.
- Your elastic instance should have the apm-agent available [Elastic setup APM docs](https://www.elastic.co/guide/en/apm/server/7.10/installing.html).

## Examples

- [React](https://github.com/thredup/rollbar-sourcemap-webpack-plugin/tree/master/examples/react)
- [Next.js](https://github.com/thredup/rollbar-sourcemap-webpack-plugin/tree/master/examples/next-js)

## Contributing

See the [Contributors Guide](/CONTRIBUTING.md)

## License

[MIT](/LICENSE.md)
