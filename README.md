# Comunica Contextify Version Query Operation Actor

[![npm version](https://badge.fury.io/js/%40comunica%2Factor-query-operation-contextify-version.svg)](https://www.npmjs.com/package/@comunica/actor-query-operation-contextify-version)
[![Build Status](https://travis-ci.org/rdfostrich/comunica-actor-query-operation-contextify-version.svg?branch=master)](https://travis-ci.org/rdfostrich/comunica-actor-query-operation-contextify-version)
[![Coverage Status](https://coveralls.io/repos/github/rdfostrich/comunica-actor-query-operation-contextify-version/badge.svg?branch=master)](https://coveralls.io/github/rdfostrich/comunica-actor-query-operation-contextify-version?branch=master)

A [Query Operation](https://github.com/comunica/comunica/tree/master/packages/bus-query-operation) actor
that detects graph-based version operations and rewrites them to operations with a version context.

This module is part of the [Comunica framework](https://github.com/comunica/comunica),
and should only be used by [developers that want to build their own query engine](https://comunica.dev/docs/modify/).

[Click here if you just want to query with Comunica](https://comunica.dev/docs/query/).

## Install

```bash
$ yarn add @comunica/actor-query-operation-contextify-version
```

## Configure

After installing, this package can be added to your engine's configuration as follows:
```text
{
  "@context": [
    ...
    "https://linkedsoftwaredependencies.org/bundles/npm/@comunica/actor-query-operation-contextify-version/^1.0.0/components/context.jsonld"  
  ],
  "actors": [
    ...
    {
      "@id": "config-sets:contextify-version.json#myContextifyVersionQueryOperator",
      "@type": "ActorQueryOperationContextifyVersion",
      "cbqo:mediatorQueryOperation": { "@id": "config-sets:sparql-queryoperators.json#mediatorQueryOperation" },
      "caqocv:Actor/QueryOperation/ContextifyVersion/baseGraphUri": "http://graph.version.",
      "caqocv:Actor/QueryOperation/ContextifyVersion/numericalVersions": true
    }
  ]
}
```

### Config Parameters

* `caqocv:Actor/QueryOperation/ContextifyVersion/baseGraphUri`: The base URI of the graph to contextify. If this is not provided, then graphs will be converted to version identifiers as-is.
* `caqocv:Actor/QueryOperation/ContextifyVersion/numericalVersions`: If versions should be parsed as integers.

## Examples

Assuming a base graph URI `http://ex/g/`.

### Version Materialization

Input query:
```sparql
SELECT * WHERE {
  GRAPH <http://ex/g/1> {
    ?s ?p ?o
  }
}
```

Output context:
```json
{
  "version": {
    "type": "version-materialization",
    "version": 1
  }
}
```

Output query:
```sparql
SELECT * WHERE {
  ?s ?p ?o
}
```

### Delta Materialization

Input query:
```sparql
SELECT * WHERE {
  GRAPH <http://ex/g/4> {
    ?s ?p ?o
  } .
  FILTER (NOT EXISTS {
    GRAPH <http://ex/g/2> {
      ?s ?p ?o
    }
  }) 
}
```

Output context:
```json
{
  "version": {
    "queryAdditions": true,
    "type": "delta-materialization",
    "versionEnd": 4,
    "versionStart": 2
  }
}
```

Output query:
```sparql
SELECT * WHERE {
  ?s ?p ?o
}
```

_Note:_ Flipping the versions would make this a _deletions_ query instead of an _additions_ query,
i.e., `queryAdditions` would be set to `false`.
