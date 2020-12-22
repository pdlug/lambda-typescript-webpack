# AWS Lambda w/ Typescript packaged with webpack

This is a simple example of using webpack to package up an AWS Lambda function written in Typescript. The configuration is pretty minimal and straightforward, it should be easy to copy into your project or modify as needed. This was inspired by [lambda-typescript-webpack-babel-starter](https://github.com/lifeomic/lambda-typescript-webpack-babel-starter) but simplified to use `ts-loader` instead of babel and to only provide packaging (no test frameworks). It also writes everything in Typescript including the webpack configuration itself. Unlike other examples out there this is just vanilla webpack. It does not use Serverless, SAM, or any other tooling.

## Getting started

Install dependencies with `yarn`:

```bash
yarn install
```

Write any Lambda functions you want in `src/`. An example of a Lambda that takes an API Gateway request in, parses the JSON payload, and returns it is provided in `src/api.ts`. Add any new functions to `entry` in `webpack.config.ts` and they'll be built.

```bash
yarn run build
```

This will compile function and output them into `dist/`. The `package` script will create a zip file or you can use the files directly. Deploy by manually uploading, [aws-cli](https://aws.amazon.com/cli/), or with a tool like [Terraform](https://terraform.io/) (my preferred option).

The handler will be `{filename}.default` if you are using the default export, so if deploying the sample API function the handler will be `api.default`.

## Structure

- `webpack.config.ts` - Webpack configuration
- `tsconfig.webpack.json` - Typescript config used for webpack itself (which must es5 target, commonjs modules, and interop with )

## Issues

I encountered a few issues using this pattern with other libraries. Primarily this was when adding in Apollo GraphQL to deploy a GraphQL server as a Lambda behind API Gateway. I've described the issues below and their solution in case it helps anyone else. Both solutions are included in the `webpack.config.ts` in the project since I see no harm in keeping them around.

The first was a series of errors like:

```text
Can't reexport the named export from non EcmaScript module (only default export is available)
```

These all referenced `.mjs` files within Apollo dependencies. The [apollo-link-state issue #302](https://github.com/apollographql/apollo-link-state/issues/302) had the solution which is to add handling of `.mjs` files in `resolve` and in `module` using the built in `javascript/auto` type.

```typescript
resolve: {
  extensions: [".mjs", ".ts", ".js"],
},
...
module: {
  rules: [
    ...
    { test: /\.mjs$/, include: /node_modules/, type: "javascript/auto" },
  ],
},
```

The other issue I encountered was an error message:

```text
Critical dependency: the request of a dependency is an expression" in iconv-loader.js
```

[webpack issue #3078](https://github.com/webpack/webpack/issues/3078#issuecomment-497051466)

```typescript
module: {
  ...
  noParse: /iconv-loader\.js/,
},
```
