# osquery

[![](https://img.shields.io/static/v1?label=godoc&message=reference&color=blue&style=flat-square)](https://godoc.org/github.com/opeious/osquery) [![](https://img.shields.io/github/license/opeious/osquery?style=flat-square)](LICENSE)


**A non-obtrusive, idiomatic and easy-to-use query and aggregation builder for the [official Go client](https://github.com/opensearch-project/opensearch-go) for [OpenSearch](https://opensearch.org/). Based on [EsQuery](https://github.com/aquasecurity/esquery)**

## Table of Contents

<!--ts-->
   * [Description](#description)
   * [Status](#status)
   * [Installation](#installation)
   * [Usage](#usage)
   * [Notes](#notes)
   * [Features](#features)
      * [Supported Queries](#supported-queries)
      * [Supported Aggregations](#supported-aggregations)
      * [Custom Queries and Aggregations](#custom-queries-and-aggregations)
   * [License](#license)
<!--te-->

## Description

`osquery` alleviates the need to use extremely nested maps (`map[string]interface{}`) and serializing queries to JSON manually. It also helps eliminating common mistakes such as misspelling query types, as everything is statically typed.

Using `osquery` can make your code much easier to write, read and maintain, and significantly reduce the amount of code you write. Wanna know how much code you'll save? just check this project's tests.

## Status

This is an early release, API may still change.

## Installation

`isquery` is a Go module. To install, simply run this in your project's root directory:

```bash
go get github.com/opeious/osquery
```

## Usage

osquery provides a [method chaining](https://en.wikipedia.org/wiki/Method_chaining)-style API for building and executing queries and aggregations. It does not wrap the official Go client nor does it require you to change your existing code in order to integrate the library. Queries can be directly built with `osquery`, and executed by passing an `*opensearch.Client` instance (with optional search parameters). Results are returned as-is from the official client (e.g. `*opensearchapi.Response` objects).

Getting started is extremely simple:

```go
package main

import (
	"context"
	"log"

	"github.com/opeious/osquery"
	"github.com/opensearch-project/opensearch-go"
)

func main() {
    // connect to an OpenSearch instance
    es, err := opensearch.NewDefaultClient()
    if err != nil {
        log.Fatalf("Failed creating client: %s", err)
    }

    // run a boolean search query
    res, err := osquery.Search().
        Query(
            osquery.
                Bool().
                Must(osquery.Term("title", "Go and Stuff")).
                Filter(osquery.Term("tag", "tech")),
        ).
        Aggs(
            osquery.Avg("average_score", "score"),
            osquery.Max("max_score", "score"),
        ).
        Size(20).
        Run(
            es,
            es.Search.WithContext(context.TODO()),
            es.Search.WithIndex("test"),
        )
        if err != nil {
            log.Fatalf("Failed searching for stuff: %s", err)
        }

    defer res.Body.Close()

    // ...
}
```

## Notes

* The library cannot currently generate "short queries". For example, whereas
  OpenSearch can accept this:

```json
{ "query": { "term": { "user": "Kimchy" } } }
```

  The library will always generate this:

```json
{ "query": { "term": { "user": { "value": "Kimchy" } } } }
```

  This is also true for queries such as "bool", where fields like "must" can
  either receive one query object, or an array of query objects. `osquery` will
  generate an array even if there's only one query object.

## Features

### Supported Queries

The following queries are currently supported:

| OpenSearch DSL          | `osquery` Function    |
| ------------------------|---------------------- |
| `"match"`               | `Match()`             |
| `"match_bool_prefix"`   | `MatchBoolPrefix()`   |
| `"match_phrase"`        | `MatchPhrase()`       |
| `"match_phrase_prefix"` | `MatchPhrasePrefix()` |
| `"match_all"`           | `MatchAll()`          |
| `"match_none"`          | `MatchNone()`         |
| `"multi_match"`         | `MultiMatch()`        |
| `"exists"`              | `Exists()`            |
| `"fuzzy"`               | `Fuzzy()`             |
| `"ids"`                 | `IDs()`               |
| `"prefix"`              | `Prefix()`            |
| `"range"`               | `Range()`             |
| `"regexp"`              | `Regexp()`            |
| `"term"`                | `Term()`              |
| `"terms"`               | `Terms()`             |
| `"terms_set"`           | `TermsSet()`          |
| `"wildcard"`            | `Wildcard()`          |
| `"bool"`                | `Bool()`              |
| `"boosting"`            | `Boosting()`          |
| `"constant_score"`      | `ConstantScore()`     |
| `"dis_max"`             | `DisMax()`            |

### Supported Aggregations

The following aggregations are currently supported:

| OpenSearch DSL          | `osquery` Function    |
| ------------------------|---------------------- |
| `"avg"`                 | `Avg()`               |
| `"weighted_avg"`        | `WeightedAvg()`       |
| `"cardinality"`         | `Cardinality()`       |
| `"max"`                 | `Max()`               |
| `"min"`                 | `Min()`               |
| `"sum"`                 | `Sum()`               |
| `"value_count"`         | `ValueCount()`        |
| `"percentiles"`         | `Percentiles()`       |
| `"stats"`               | `Stats()`             |
| `"string_stats"`        | `StringStats()`       |
| `"top_hits"`            | `TopHits()`           |
| `"terms"`               | `TermsAgg()`          |

### Supported Top Level Options

The following top level options are currently supported:

| OpenSearch DSL          | `osquery.Search` Function              |
| ------------------------|--------------------------------------- |
| `"highlight"`           | `Highlight()`                          |
| `"explain"`             | `Explain()`                            |
| `"from"`                | `From()`                               |
| `"postFilter"`          | `PostFilter()`                         |
| `"query"`               | `Query()`                              |
| `"aggs"`                | `Aggs()`                               |
| `"size"`                | `Size()`                               |
| `"sort"`                | `Sort()`                               |
| `"source"`              | `SourceIncludes(), SourceExcludes()`   |
| `"timeout"`             | `Timeout()`                            |

#### Custom Queries and Aggregations

To execute an arbitrary query or aggregation (including those not yet supported by the library), use the `CustomQuery()` or `CustomAgg()` functions, respectively. Both accept any `map[string]interface{}` value.

## License

This library is distributed under the terms of the [Apache License 2.0](LICENSE).
