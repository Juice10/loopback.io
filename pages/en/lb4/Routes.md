---
lang: en
title: 'Routes'
keywords: LoopBack 4.0, LoopBack 4
tags:
sidebar: lb4_sidebar
permalink: /doc/en/lb4/Routes.html
summary:
---

## Overview

A `Route` is the mapping between your API specification and an Operation (JavaScript implementation). It tells LoopBack which function to `invoke()` given an HTTP request.

The `Route` object and its associated types are provided as a part of the
[(`@loopback/rest`)](https://github.com/strongloop/loopback-next/blob/master/packages/rest) package.

## Operations

Operations are JavaScript functions that accept Parameters. They can be implemented as plain JavaScript functions or as methods in [Controllers](Controllers.html).

```js
// greet is a basic operation
function greet(name) {
  return `hello ${name}`;
}
```

## Parameters

In the example above, `name` is a Parameter. Parameters are values, usually parsed from a `Request` by a `Sequence`, passed as arguments to an Operation. Parameters are defined as part of a `Route` using the OpenAPI specification. They can be parsed from the following parts of the `Request`:

 - `body`
 - `query` string
 - `header`
 - `path` (url)

## Creating REST Routes

The example below defines a `Route` that will be matched for `GET /`. When the `Route` is matched, the `greet` Operation (above) will be called. It accepts an OpenAPI [OperationObject](https://github.com/OAI/OpenAPI-Specification/blob/0e51e2a1b2d668f434e44e5818a0cdad1be090b4/versions/2.0.md#operationObject) which is defined using `spec`.
The route is then attached to a valid server context running underneath the
application.
```js
const app = new Application();

const spec = {
  parameters: [{name: 'name', in: 'query', type: 'string'}],
  responses: {
    '200': {
      description: 'greeting text',
      schema: {type: 'string'},
    }
  }
};

(async function start() {
  const server = await app.getServer(RestServer);
  const route = new Route('get', '/', spec, greet);
  server.route(route);
  await app.start();
})();

```

## Declaring REST Routes with API specifications

Below is an example [Open API Specification](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md#swagger-object) that defines the same operation as the example above. This a declarative approach to defining operations. The `x-operation` field in the example below references the handler JavaScript function for the API operation, and should not be confused with `x-operation-name`, which is a string for the Controller method name.

```js

const server = await app.getServer(RestServer);
const spec = {
  basePath: '/',
  paths: {
    '/': {
      get: {
        'x-operation': greet,
        parameters: [{name: 'name', in: 'query', type: 'string'}],
        responses: {
          '200': {
            description: 'greeting text',
            schema: {type: 'string'},
          }
        }
      }
    }
  }
};

server.api(spec);
```

## Invoking operations using Routes

This example breaks down how `Sequences` determine and call the matching operation for any given request.

```js
class MySequence extends DefaultSequence {
  async handle(request, response) {
    // find the route that matches this request
    const route = this.findRoute(request);

    // params is created by parsing the request using the route
    const params = this.parseParams(request, route);

    // invoke() uses both route and params to execute the operation specified by the route
    const result = await this.invoke(route, params);

    await this.send(response, result);
  }
}
```
