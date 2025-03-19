+++
date = '2025-03-17T15:32:54+01:00'
draft = true
title = 'How I Autogenerate TypeScript for API Contracts'
+++

Writing types for the backend response in typescript is tedious.

That's why in my team we switched to `Open Api Schema` and generate types from that.

## The flow

### 1. Backend provides the schema

Backend has two options:

1. have the yaml schema as a file
2. have swagger or something similar.

In the first case we as frontend go get this file and put in our repo.

In the second case if it's swagger we get the json schema first and then convert to yaml with any convert tool
![swagger pic](/swagger_json.png)

### 2. Setup the generator tool

We use [@openapitools/openapi-generator-cli](https://github.com/OpenAPITools/openapi-generator-cli) and the best way to set it up we found is:

1. install

```bash
npm i -D @openapitools/openapi-generator-cli
```

2. create a config file `openapitools.json`

```js
{
  "$schema": "./node_modules/@openapitools/openapi-generator-cli/config.schema.json",
  "generator-cli": {
    "version": "7.9.0",
    "generators": {
      "file1-openapi": {
        "inputSpec": "./openapi/schemas/file1-openapi.yaml",
        "generatorName": "typescript-axios",
        "output": "./openapi/generated/file1-openapi"
      },
      "file2-openapi": {
        "inputSpec": "./openapi/schemas/file2-openapi.yaml",
        "generatorName": "typescript-axios",
        "output": "./openapi/generated/file2-openapi"
      },
    }
  }
}
```

3. Run `npx @openapitools/openapi-generator-cli generate`

## Example

for the schema file
file1.yaml

```yml
openapi: 3.0.0
info:
  title: test api
  version: 1.0.0
paths:
  /api/example:
    get:
      responses:
        "200":
          description: OK
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/example"
components:
  schemas:
    example:
      type: object
      properties:
        field1:
          type: string
        field2:
          type: string
      required:
        - field1
        - field2
      additionalProperties: false
```

we generate some files where we have most importantly:

```js
export interface Example {
  field1: string;
  field2: string;
}
```

and

```js
export class DefaultApi extends BaseAPI {
    public apiExampleGet(options?: RawAxiosRequestConfig) {
        return DefaultApiFp(this.configuration).apiExampleGet(options).then((request) => request(this.axios, this.basePath));
    }
}
```

The first is our type and the second is our api call.

We can call `const response = new DefaultApi().apiExampleGet()` and have the response that is already typed. How convenient!

One thing I found myself doing is wrapping this call in my own class where I do parameter validation, throwing sooner if parameters are not expected and passing a custom endpoint

```js
class OpenApiService {
  private static customConfig = new Configuration({
    basePath: process.env.NEXT_PUBLIC_apiUrl,
  })
  static async apiExampleGet(
    superParameter?: string,
  ) {
    if (!superParameter) throw new Error('superParameter is required')

    const apiInstance = new DefaultApi(this.customConfig)
    const response = await apiInstance.exampleget(
      superParameter,
    )
    return response.data
  }
}
```

### Conclusion

All this saves us a lot of time in our team by not having to write this boilerplate ourselves and being able to update and have our types in sync when the schema changes.

### Backend appendix

our flow is that we agree in frontend and backend on the response and then backend writes first the schema by hand and then generates its c# classes by a similar tool called [nswag](https://github.com/RicoSuter/NSwag)

This approach works but there are no blockers to write the backend types/code first and then generate the schema from that.
