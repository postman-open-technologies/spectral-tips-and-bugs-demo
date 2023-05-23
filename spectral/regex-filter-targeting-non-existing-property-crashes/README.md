# Spectral crashes when a regex filter targets a non existing property

Spectral crashes when a given path contains a regex filter on a specific property `[?(@.someProperty.match(/regex/))]` and traverse an object that doesn't contain it.

# Issue

The following rules looks for any schema under components schema having a type starting by "arr". 

- Unit test: `npm test -- --rule regex-crash`
- Spectral run: `npm run spectral-run-regex-crash`

```
rules:
  regex-crash:
    formats: [oas3_1]
    given:
      - $.components.schemas..[?(@.type.match(/^arr/))]
    then:
      - field: maxItems
        function: defined

```

Linting the following document crashes Spectral with a `Cannot read properties of undefined (reading 'match')`error:

```
openapi: "3.1.0"
info:
  title: An API
  version: "1.0"
components:
  schemas:
    SchemaObject:
      properties:
        aProperty:
          type: string
```

# Workaround

Adding `@.name != undefined` in the filter fixes the issue.

- Unit test: `npm test -- --rule regex-work-around`
- Spectral run: `spectral-run-regex-workaround`

```
rules:
  regex-work-around:
    given:
      - $..[?(@.type != undefined && @.type.match(/^arr/))]
    then:
      - field: maxItems
        function: defined
```