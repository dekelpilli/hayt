# Hayt [![Build Status](https://secure.travis-ci.org/mpenet/hayt.png?branch=master)](http://travis-ci.org/mpenet/hayt) [![cljdoc badge](https://cljdoc.xyz/badge/cc.qbits/hayt)](https://cljdoc.xyz/d/cc.qbits/hayt/CURRENT)

CQL3 DSL for Clojure.

## What?

Hayt is a thin query DSL for CQL.

It works in 2 simple steps, first the query api generates a map (AST)
representation of the query, allowing you to compose/modify it at
will, then there's a compilation step that will generate a raw string
for you to use in prepared statements or normal queries.

Both are decoupled, so you can just use the AST directly and make your
own api on top of it if that's what you like.

## What's in the box?

* **Complete CQL 3.1.1+ coverage** including some features in Cassandra
  trunk, **DDL**, **CQL Functions**, **counter** , **triggers** and
  **collections** operations
* Support for both **Raw queries** and **Prepared Statements** generation
* **Great performance** (lots of transducing and fiddling with StringBuilder)
* **Extensive test coverage**
* Decoupled query compiler, allowing you to **build your own DSL** in minutes
* Highly **composable** using simple maps or the functions provided
* Extensible **Clojure data types support**
* Constantly **kept up to date** with Cassandra changes, almost daily
* No (exposed) macros

## Installation

Please check the
[Changelog](https://github.com/mpenet/hayt/blob/master/CHANGELOG.md) first
if you are upgrading.

[![Clojars Project](https://img.shields.io/clojars/v/cc.qbits/hayt.svg)](https://clojars.org/cc.qbits/hayt)

Note: while in beta the API is still subject to changes.

## Usage

This should be familiar if you know Korma or ClojureQL.
One of the major differences is that Hayt doesn't expose macros.


```clojure

(use 'qbits.hayt)

(select :foo
        (where {:bar 2}))

(update :some-table
         (set-columns {:bar 1
                       :baz (inc-by 2)})
         (where [[= :foo :bar]
                 [> :moo 3]
                 [:> :meh 4]
                 [:in :baz [5 6 7]]]))
```

All these functions do is generate maps, if you want to build your own
DSL on top of it or use maps directly, feel free to do so.

```clojure
(select :users
        (columns :a :b)
        (where [[= :foo :bar]
                [> :moo 3]
                [:> :meh 4]
                [:in :baz [5 6 7]]])})

;; generates the following

>> {:select :users
    :columns [:a :b]
    :where [[= :foo :bar]
            [> :moo 3]
            [:> :meh 4]
            [:in :baz [5 6 7]]]}
```

Since Queries are just maps they are composable using the usual `merge`
`into` `assoc` etc.

```clojure
(def base (select :foo (where {:foo 1})))

(merge base
       (columns :bar :baz)
       (where {:bar 2})
       (order-by [:bar :asc])
       (using :ttl 10000))

```

To compile the queries just use `->raw`

```clojure
(->raw (select :foo))
> "SELECT * FROM foo;"


(->raw {:select :foo :columns [:a :b]})
> "SELECT a, b FROM foo;"

;; or if you want full control of what's prepared you can use
;; `cql-raw` with ->raw compilation:


;; here `?` is an alias to `(cql-raw "?")`

(->raw (select :foo (where {:bar 1 :baz ?})))
> "SELECT * FROM foo WHERE bar = 1 AND baz = ?;"


;; and named parameters using keywords

(->raw (select :foo (where {:bar 1 :baz :named)}))
> "SELECT * FROM foo WHERE bar = 1 AND baz = :named;"
```

When compiling with `->raw` we take care of the encoding/escaping
required by CQL. This process is also open via the
`qbits.hayt.CQLEntities` protocol for both values and identifiers. We
also supply a joda-time codec for you to use if you require to in
`qbits.hayt.codec.joda-time`. This codec is a good example of how to
handle custom type encoding.

If you are curious about what else it can do, head to the
[codox documentation](http://mpenet.github.com/hayt/codox/qbits.hayt.html)
or the
[tests](https://github.com/mpenet/hayt/blob/master/test/qbits/hayt/core_test.clj).

## License

Copyright @[Max Penet](https://twitter.com/mpenet)

Distributed under the Eclipse Public License, the same as Clojure.
