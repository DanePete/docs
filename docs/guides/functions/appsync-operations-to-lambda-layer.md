---
title: Exporting AppSync operations to a Lambda layer for easy reuse
description: How to export your AppSync operations to a Lambda layer for easy reuse in your Lambda functions
---

When you need to call your AppSync API from a Lambda function, you can export your AppSync operations generated by Amplify to a Lambda layer. This reduces the amount of duplicated code, and allows you to interact with the API from Lambda functions across multiple projects.

## Set up your layer

Start by creating the layer that will contain your AppSync operations.

```sh
$ amplify add function

? Select which capability you want to add:
❯ Lambda layer (shared code & resource used across functions)

? Provide a name for your Lambda layer: <your-lambda-layer-name>

? Choose the runtime that you want to use:
❯ NodeJS

✅ Lambda layer folders & files created:
amplify/backend/function/appsyncOperations
```

In the `./amplify/backend/function/appsyncOperations/opt` folder add the following helper library. The library exports a `request` function that you can use to send queries or mutations to your AppSync API. You can specify an API KEY to include the `x-api-key` header in the request. Otherwise the request is signed using SigV4 and the invoked Lambda function's credentials.

```javascript
// amplify/backend/function/appsyncOperations/opt/appSyncRequest.js
const https = require('https')
const AWS = require('aws-sdk')
const urlParse = require('url').URL
const region = process.env.REGION

/**
 *
 * @param {Object} queryDetails the query, operationName, and variables
 * @param {String} appsyncUrl url of your AppSync API
 * @param {String} apiKey the api key to include in headers. if null, will sign with SigV4
 */
exports.request = (queryDetails, appsyncUrl, apiKey) => {
  const req = new AWS.HttpRequest(appsyncUrl, region)
  const endpoint = new urlParse(appsyncUrl).hostname.toString()

  req.method = 'POST'
  req.path = '/graphql'
  req.headers.host = endpoint
  req.headers['Content-Type'] = 'application/json'
  req.body = JSON.stringify(queryDetails)

  if (apiKey) {
    req.headers['x-api-key'] = apiKey
  } else {
    const signer = new AWS.Signers.V4(req, 'appsync', true)
    signer.addAuthorization(AWS.config.credentials, AWS.util.date.getDate())
  }

  return new Promise((resolve, reject) => {
    const httpRequest = https.request({ ...req, host: endpoint }, (result) => {
      result.on('data', (data) => {
        resolve(JSON.parse(data.toString()))
      })
    })

    httpRequest.write(req.body)
    httpRequest.end()
  })
}
```

## Generate compatible code for your layer

Install the Babel dependencies to transpile the GraphQL javascript files generated by the Amplify codegen

```sh
npm i --save-dev @babel/core @babel/cli @babel/preset-env
```

Add a Babel configuration file that specifies the compilation target. The configuration specifies that the only target is NodeJS 12. In `./babel.config.json`:

```json
{
  "presets": [
    [
      "@babel/env",
      {
        "targets": {
          "node": "12"
        }
      }
    ]
  ]
}
```

Add a command to the `scripts` in your `package.json` that updates your layer with the latest codegen:

```json
{
  "scripts": {
    "updateAppsyncOperations": "amplify api gql-compile && amplify codegen && babel src/graphql --config-file ./babel.config.json -d ./amplify/backend/function/<your-lambda-layer-name>/opt/graphql/"
  }
}
```

Make sure to replace `<your-lambda-layer-name>` with the actual name of your Lambda layer.

You can run this command after you've modified your schema to update the codegen and your layer:

```sh
npm run updateAppsyncOperations
```

The `opt` directory of in your layer now looks like this:

```sh
amplify/backend/function/appsyncOperations/opt
├── appsyncRequest.js
└── graphql
    ├── mutations.js
    ├── queries.js
    └── subscriptions.js
```

You can push your changes to the cloud with `amplify push`.

You can now assign the layer to Lambda functions to easily call your AppSync API. Here's how you can leverage it in a Lambda function that has IAM permissions to call your AppSync API.

```javascript
const appsyncUrl = process.env.API_AMPLIFYLAYERGUIDE_GRAPHQLAPIENDPOINTOUTPUT
const apiKey = process.env.API_AMPLIFYLAYERGUIDE_GRAPHQLAPIKEYOUTPUT

const { request } = require('/opt/appsyncRequest')
const { createTodo } = require('/opt/graphql/mutations')
const { listTodos } = require('/opt/graphql/queries')

exports.handler = async (event) => {
  try {
    // create a TODO with AWS_IAM auth
    var result = await request(
      {
        query: createTodo,
        variables: {
          input: {
            name: 'new todo',
            description: 'the first',
          },
        },
      },
      appsyncUrl
    )
    console.log('iam results:', result)
    
    // Now, retrieve all TODOs using API_KEY auth
    result = await request(
      {
        query: listTodos,
        operationName: 'ListTodos',
      },
      appsyncUrl,
      apiKey
    )
    console.log('api key results', result)
  } catch (error) {
    console.log(error)
  }
}
```