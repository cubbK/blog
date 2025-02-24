+++
date = '2025-02-19T15:32:54+01:00'
draft = true
title = 'Openapi Schema in Frontend'
+++

Scenario:

- Frontend uses typescript
- Frontend uses an rest endpoint and fetches data
- The data received is _any_
- Frontend manually types the response

But doing it manually can be avoided!

Introducing openapi schema!

`npx @openapitools/openapi-generator-cli generate`
