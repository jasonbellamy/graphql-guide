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

> Mutations and subscriptions have different semantics (mutations cause side effects, subscriptions return more than the initial result), but all three types of operations are identical in that they return a "graph" of results. In this article we'll focus on queries, and consider the other operations in later articles, but keep in mind that everything that applies to queries apply to the other two operations also.

<h2 id="basics">Query Basics</h2>

A query starts at one of the fields of the base `query` type in the schema (conventionally called `Query`).

So if the schema looks like:

```
type Query {
  me: Profile
  user(id: ID): Profile
  publicFeed: [Activity]
}

schema {
  query: Query
}
```

The query must start at `me`, `user` or `publicFeed`.

From there, the query must make a selection of one or more fields for each type that it encounters. For instance, the `me` query above chooses the field `feed` off of the `Profile` object for the initial `me` field.

If the field it chooses it a scalar, the query stops at that point. If the field is another object type (or an interface), the query must again choose one or more fields within that type, denoted by `{ }`.

So if the first choice was `me` (return a `Profile`), and then the query chooses `name` and `feed`, as name is a scalar, the query stops, and as feed is an object type, the query must choose sub-fields:

```
{
  me {
    name
    feed {
      name
    }
  }
}
```

> Note that when the field is an [array type](schemas.html#arrays), you choose fields off the wrapped type of the array (and each element returned in the array will have those fields).

> When the field is an [interface](schemas.html#interface), you can only choose fields from the interface. As the `Profile`'s `feed` field has type `[Activity]`, we can only choose `name` as a subfield, although we can use (optionally inline) fragments to get other fields as we'll see [below](fragments).

> When the field is a union, we *must* use (optionally inline) fragments to select further.

<h3 id="variables">Variables</h3>

When the a field is [parameterized](schema.html#fields), the query must specify a value for the parameter. A constant value can be provided:

```
{
  user(id: "23") {
    name
  }
}
```

You can construct your query string dynamically in and hardcode in such constant values, but typically you would instead use a *static* query (i.e. the same query for different situations) and pass *variables* into it.

> When you are using variables in you query, you can't use the [single query shorthand](#documents).

```

```

<h3 id="interfaces">Interfaces and Unions</h3>

<h2 id="execution">Executing Queries</h2>

<h3 id="network-layer">Network Layer</h3>

<h3 id="query-resolution">Query Resolution</h3>

<h2 id="advanced">Advanced Queries</h2>

<h3 id="fragments">Fragments</h3>

<h3 id="inline-fragments">Inline Fragments</h3>

<h3 id="directives">Directives</h3>
