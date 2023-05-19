# Recursive descent ".." ignores first level #

In Spectral, a given path containing `..[<filter>]` will only find elements beyond the first level.

# Issue

A Spectral rule using a path such as `$.components.schemas.*..[?(@.type=="array")]` to detect arrays in reusable schemas in an OpenAPI document will not detect first level arrays.

```
openapi: "3.1.0"
info:
  title: An API
  version: "1.0"
components:
  schemas:
    SchemaArray: # Not found
      type: array
      items:
        type: object
    SchemaObject:
      type: object
      properties:
        aPropertyArray: # Found 
          type: array
```

A rule using similar path without filter like `$.components.schemas.*..type` will detect all elements.

```
openapi: "3.1.0"
info:
  title: An API
  version: "1.0"
components:
  schemas:
    SchemaBoolean:
      type: boolean # Found
    SchemaObject:
      properties:
        aBooleanProperty:
          type: boolean # Found
```
# Work around

A possible work around consists in adding a path specifically targeting first level.

Bug:

```
given:
  - $.some.path..[<filter>]
```

Work around:

```
given:
  - $.some.path..[<filter>]
  - $.some.path.[<filter>]
```

# Unit tests

## Unit test for rule demonstrating the bug

Run unit test only for bug: `npm run recursive-descent-ignores-first-level-bug-unit-test`

```
  Validating Spectral Rulesets
    🔬 Checking test documents (spectral/recursive-descent-ignores-first-level/**/*.spectral-test.yaml) are valid
      ✔ must have found and aggregated runnable tests
      ✔ must find no test invalid against schema
      ✔ must find no test targeting non-existing ruleset files
    🤖 Checking Spectral Wrapper has been loaded
      ✔ no error when loading ruleset with Spectral
    🗂  Testing ruleset ../recursive-descent-ignores-first-level.spectral-v6.yaml
      📏 Testing rule [recursive-descent-and-filter-bug]
        ✔ must target format(s) openapi [3.1]
        ✔ must have severity undefined
        ✔ must be recommended
        Testing rule recursive-descent-and-filter-bug given
          1) must find first level array (openapi 3.1)
          ✔ must find deep array (openapi 3.1)


  8 passing (198ms)
  1 failing

  1) Validating Spectral Rulesets
       🗂  Testing ruleset ../recursive-descent-ignores-first-level.spectral-v6.yaml
         📏 Testing rule [recursive-descent-and-filter-bug]
           Testing rule recursive-descent-and-filter-bug given
             must find first level array (openapi 3.1):

      AssertionError [ERR_ASSERTION]: found paths do not match expected ones
      + expected - actual

      -[]
      +[
      +  [
      +    "components"
      +    "schemas"
      +    "SchemaArray"
      +  ]
      +]
```

## All unit tests

Run all unit tests (bug demo, work around, no problem without filter): `recursive-descent-ignores-first-level-all-unit-tests`

# Spectral run with sample documents

Run on sample document: `npm run recursive-descent-ignores-first-level-spectral-run`

The `recursive-descent-and-filter-bug` detects only 1 issue instead of 2 (as `recursive-descent-and-filter-work-around-1` and `recursive-descent-and-filter-work-around-2` do).

```
recursive-descent-ignores-first-level/samples/recursive-descent-and-filter.openapi.yaml
  7:17  warning  recursive-descent-and-filter-work-around-1  "SchemaArray.maxItems" property must be defined     components.schemas.SchemaArray
  7:17  warning  recursive-descent-and-filter-work-around-2  "SchemaArray.maxItems" property must be defined     components.schemas.SchemaArray
 14:24  warning  recursive-descent-and-filter-bug            "aPropertyArray.maxItems" property must be defined  components.schemas.SchemaObject.properties.aPropertyArray
 14:24  warning  recursive-descent-and-filter-work-around-1  "aPropertyArray.maxItems" property must be defined  components.schemas.SchemaObject.properties.aPropertyArray
 14:24  warning  recursive-descent-and-filter-work-around-2  "aPropertyArray.maxItems" property must be defined  components.schemas.SchemaObject.properties.aPropertyArray

✖ 5 problems (0 errors, 5 warnings, 0 infos, 0 hints)
```
