[[query-dsl-intro]]
=== Query DSL

The query DSL is a flexible, expressive search ((("Query DSL")))language that Elasticsearch
uses to expose most of the power of Lucene through a simple JSON interface. It
is what you should be using to write your queries in production. It makes your
queries more flexible, more precise, easier to read, and easier to debug.

To use the Query DSL, pass a query((("query parameter"))) in the `query` parameter:

[source,js]
--------------------------------------------------
GET /_search
{
    "query": YOUR_QUERY_HERE
}
--------------------------------------------------

The _empty search_&#x2014;`{}`&#x2014;is ((("empty search", "equivalent to match_all query clause")))functionally equivalent to using the
`match_all` query clause, which,((("match_all query clause"))) as the name suggests, matches all documents:

[source,js]
--------------------------------------------------
GET /_search
{
    "query": {
        "match_all": {}
    }
}
--------------------------------------------------
// SENSE: 054_Query_DSL/60_Empty_query.json

==== Structure of a Query Clause

A query clause typically((("Query DSL", "structure of a query clause"))) has this structure:

[source,js]
--------------------------------------------------
{
    QUERY_NAME: {
        ARGUMENT: VALUE,
        ARGUMENT: VALUE,...
    }
}
--------------------------------------------------


If it references one particular field, it has this structure:

[source,js]
--------------------------------------------------
{
    QUERY_NAME: {
        FIELD_NAME: {
            ARGUMENT: VALUE,
            ARGUMENT: VALUE,...
        }
    }
}
--------------------------------------------------



For instance, you can use a `match` query clause((("match query"))) to find tweets that
mention `elasticsearch` in the `tweet` field:

[source,js]
--------------------------------------------------
{
    "match": {
        "tweet": "elasticsearch"
    }
}
--------------------------------------------------


The full search request would look like this:

[source,js]
--------------------------------------------------
GET /_search
{
    "query": {
        "match": {
            "tweet": "elasticsearch"
        }
    }
}
--------------------------------------------------
// SENSE: 054_Query_DSL/60_Match_query.json

==== Combining Multiple Clauses

_Query clauses_ are simple building blocks((("Query DSL", "combining multiple clauses"))) that can be combined with each
other to create complex queries. Clauses can be as follows:

* _Leaf clauses_ (like the `match` clause) that((("leaf clauses"))) are used to
  compare a field (or fields) to a query string.

* _Compound_ clauses that are used ((("compound query clauses")))to combine other query clauses.
  For instance, a `bool` clause((("bool clause"))) allows you to combine other clauses that
  either `must` match,  `must_not` match, or `should` match if possible:

[source,js]
--------------------------------------------------
{
    "bool": {
        "must":     { "match": { "tweet": "elasticsearch" }},
        "must_not": { "match": { "name":  "mary" }},
        "should":   { "match": { "tweet": "full text" }}
    }
}
--------------------------------------------------
// SENSE: 054_Query_DSL/60_Bool_query.json


It is important to note that a compound clause can combine _any_ other
query clauses, including other compound clauses. This means that compound
clauses can be nested within each other, allowing the expression
of very complex logic.

As an example, the following query looks for emails that contain
`business opportunity` and should either be starred, or be both in the Inbox
and not marked as spam:

[source,js]
--------------------------------------------------
{
    "bool": {
        "must": { "match":      { "email": "business opportunity" }},
        "should": [
             { "match":         { "starred": true }},
             { "bool": {
                   "must":      { "folder": "inbox" }},
                   "must_not":  { "spam": true }}
             }}
        ],
        "minimum_should_match": 1
    }
}
--------------------------------------------------


Don't worry about the details of this example yet; we will explain in
full later. The important thing to take away is that a compound query
clause can combine multiple clauses--both leaf clauses and other
compound clauses--into a single query.
