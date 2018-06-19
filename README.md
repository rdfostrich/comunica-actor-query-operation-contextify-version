# Comunica Contextify Version Query Operation Actor

[![npm version](https://badge.fury.io/js/%40comunica%2Factor-query-operation-contextify-version.svg)](https://www.npmjs.com/package/@comunica/actor-query-operation-contextify-version)
[![Build Status](https://travis-ci.org/rdfostrich/comunica-actor-query-operation-contextify-version.svg?branch=master)](https://travis-ci.org/rdfostrich/comunica-actor-query-operation-contextify-version)
[![Coverage Status](https://coveralls.io/repos/github/rdfostrich/comunica-actor-query-operation-contextify-version/badge.svg?branch=master)](https://coveralls.io/github/rdfostrich/comunica-actor-query-operation-contextify-version?branch=master) [![Greenkeeper badge](https://badges.greenkeeper.io/rdfostrich/comunica-actor-query-operation-contextify-version.svg)](https://greenkeeper.io/)

A comunica Contextify Version Query Operation Actor.

This detects graph-based version operations and rewrites them to operations with a version context.

This module is part of the [Comunica framework](https://github.com/comunica/comunica).

## Install

```bash
$ yarn add @comunica/actor-query-operation-contextify-version
```

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
