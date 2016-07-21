---
title: Queries
order: 2
---

Queries are the key part of of GraphQL, and perhaps the most intuitive. When you write a GraphQL query, you are describing the shape of the graph of data you want back--with the restriction it must be valid as per the server's [schema](schemas.html).

So this query:

```graphql
{
  user {
    name
    avatar
    friends {
      name
    }
  }
}
```

Becomes something like:

```
{
  "data": {
    "user": {
      "name": "Tom Coleman",
      "avatar": "http://some.cdn/avatar"
      "friends": [
        {
          "name": "Sashko Stubailo"
        }
      ]
    }
  }
}
```

For the mathematically inclined, if you think of the type system entailed in the schema as a *directed graph*, then a query is simply a set of walks through the graph all starting at a common point (a single [operation entry point](schemas.html#operations)). Such a collection of walks then forms a tree (or the "graph" above).

<h2 id="documents">Query Documents</h2>

When you send operations (queries, mutations or subscriptions) to the GraphQL server, you send a group of one or more in a single GraphQL "query document". As each operation will return it's own set of results (and possibly errors), you must name each operation and provide its operation type.

```
query Feed {
  me {
    feed {
      name
      likeCount
    }
  }
}

mutation likeActivity {
  item(id: 3) {
    name
    likeCount
  }
}
```

However, if you are just sending a single operation without variables or directives, you can leave out the name, and if it is a `query`, you can leave that keyword out too:

```
{
  me {
    feed {
      name
      likeCount
    }
  }
}
```

We'll tend to write queries in this form throughout this guide.

<h2 id="basics">Query Basics</h2>

A query starts at a

<h3 id="variables">Variables</h3>

<h3 id="interfaces">Interfaces</h3>

<h2 id="execution">Executing Queries</h2>

<h3 id="network-layer">Network Layer</h3>

<h3 id="query-resolution">Query Resolution</h3>

<h2 id="advanced">Advanced Queries</h2>

<h3 id="fragments">Fragments</h3>

<h3 id="inline-fragments">Inline Fragments</h3>

<h3 id="directives">Directives</h3>
