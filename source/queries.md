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

<h2 id="operations">Operations</h2>

GraphQL consists of three kinds of operation that you can perform against your server:
  - a **query** to fetch a fixed set of data.
  - a **mutation** to modify something and get back a changed set of data.
  - and a **subscription** to subscribe to changes to a set of data.

> Subscriptions are still a draft part of the specification, but we'll document them here.

When you create your schema, the starting points are the ["operation types"](schemas.html#operations) that define the starting points of the three kinds of operations.

A GraphQL operation begins with the kind of operation (`query`, `mutation` or `subscription`), a name for this particular operation, followed by any variables or directives that apply to this operation (see [below](#variables)), and then the shape of the data to be returned:

```
query myFeed {
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

> The name of the operation is optional, however it's generally useful for debugging purposes.

If you are just sending a single `query` operation (a query) without a name, variables or directives, you can leave out the `query` keyword too:

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

We'll tend to write queries in this form throughout this guide, for simplicity.

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

So if the first choice was `me` (return a `Profile`), and then the query chooses `name` and `feed`, as `name` is a scalar, the query stops, and as `feed` is an object type, the query must choose sub-fields:

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

> Note that when the field is an [array type](schemas.html#arrays), you choose fields from they type that the array wraps (and each element returned in the array will have those fields).

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

> When you are using variables in you query, you can't use the [single query shorthand](#operations).

```
query getUser($id: ID){
  user(id: $id) {
    name
  }
}
```

Then, when you pass the query to the GraphQL server, you provide a value for `$id` also. Reusing parameterized queries like this enables the server to do advanced things like caching them.

<h2 id="execution">Executing Queries</h2>

When thinking about GraphQL queries, it's useful to keep in mind, at least abstractly, how they are evaluated by the server that is servicing the queries. Although the details of exactly how it works differs by [implementation](servers.html), the basic approach of resolving queries is dictated by the specification.

<h3 id="network-layer">Network Layer</h3>

Although it's not strictly part of the GraphQL spec, typically what happens when a query is made is that request is made over some kind of network connection, usually HTTP (although when dealing with [subscriptions](subscriptions), different, bidirectional transports may be involved).

An app's user interface will kick off a query on a GraphQL client, which will construct a [GraphQL operation](#operations) and make a request to the server's GraphQL endpoint. The server will take that request, resolve it, and then return the data on the response to the request. Most implementations currently use the JSON format for that response, which is why use that in our examples, although that is by no means required.

<h3 id="query-resolution">Query Resolution</h3>

When the server receives the operation, it executes the separate queries contained within in. Queries can be executed in any order or in parallel, but mutations must be executed serially.

To execute a query, a server must recursively *resolve* each field of each object. When you write a schema, you provide a set of *resolvers* alongside your type definitions. Those resolvers codify how to fetch the value for any given field of a type.

If the field is a scalar, then the return value of the resolver is returned to the requester. If it's a object field, the result is provided to it type's field resolvers.

So, if our schema looks like:

```
type Profile {
  name: String
  friends: [Profile]
}
type Query {
  me: Profile
}
schema {
  query: Query
}
```

We would need to provide:
  1. a resolver for the `me` field on `Query`.
  2. a resolver for the `name` field, which would have a `Profile` object (say `profile`) to work from (and if it just returns `profile.name`, in many server implementations can be left out)
  3. a resolver for the `friends` field on a given `Profile` object.

These resolvers are the point in which you GraphQL server will talk to your database, service layer, or other data store.

<h2 id="advanced">Advanced Queries</h2>

The concepts above are enough to get started writing queries, but there a few more advanced techniques that you'll find useful as you explore GraphQL further.

<h3 id="fragments">Fragments</h3>

If you find your queries re-use the same set of fields on the same type, you can shorten your queries by defining the common set of fields in a *fragment*:

So, if you were doing a nested `friends` query, instead of:
```
{
  user {
    name
    avatar
    friends {
      name
      avatar
      friends {
        name
        avatar
      }
    }
  }
}
```

You can instead write:

```
{
  user {
    ...profileFragment
    friends {
      ...profileFragment
      friends {
        ...profileFragment
      }
    }
  }
}
fragment profileFragment on Profile {
  name
  avatar
}
```

Fragments are defined for a single type (object, union or interface). The `...fragmentName` inclusion is called a *fragment spread* and is analogous to [object spread](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Spread_operator) in JavaScript.

> Note that fragments can include spreads of other fragments, but you *cannot* form cycles. It would be tempting to include `friends { ...profileFragment }` as a selection in `profileFragment`, but it would form an infinite loop and an arbitrarily large query.

<h3 id="fragments-for-interfaces">Fragments for Interfaces</h3>

When selecting a field that is an interface type, you are only allowed to sub-select fields that are defined on the interface. This is quite restrictive when the implementors of the interface have other fields:

```
interface Activity {
  title: String
}

type Comment implements Activity {
  title: String
  commenter: Profile
}

type Page implements Activity {
  title: String
  url: String
  image: String
}

type Query {
  feed: [Activity]
}

schema {
  query: Query
}
```

Without fragments, when querying this schema, you can only ask for the `title` of the items in your feed. However, if you use fragments, you can get all the information available:

```
{
  feed {
    title
    ...commentFragment
    ...pageFragment
  }
}

fragment commentFragment on Comment {
  commenter {
    name
  }
}

fragment pageFragment on Page {
  url
  image
}
```

<h3 id="fragments-for-unions">Fragments for Unions</h3>

Unions suffer from a similar problem to interfaces above: when you hit a union in your query, there are no guaranteed common fields to choose from! This, combined with the fact that you have to choose at least on field of "non-leaf" types, means you actually *have* to use fragments to query unions.

So, for example, if `Activity` was instead a union type:

```
type Comment {
  text: String
  commenter: Profile
}

type Page {
  title: String
  url: String
  image: String
}

union Activity = Comment | Page
```

Then you would need to write:

```
{
  feed {
    ...commentFragment
    ...pageFragment
  }
}

fragment commentFragment on Comment {
  text
  commenter {
    name
  }
}

fragment pageFragment on Page {
  title
  url
  image
}
```

<h3 id="inline-fragments">Inline Fragments</h3>

If you are using fragments solely for type choices (i.e. for [interfaces](#fragments-for-interfaces) or [unions](#fragments-for-unions)) and never reusing them, there is a handy inline shorthand that avoids needing to define the fragment separately from where you use it. So for our union example above, we could write:

```
{
  feed {
    ... on Comment {
      text
      commenter {
        name
      }
    }
    ... on Page {
      title
      url
      image
    }
  }
}
```

<h3 id="directives">Directives</h3>

Directives are an advanced feature of schemas that allow extensibility and adding features without necessarily having to define new syntax. They are a way of annotating fields or queries to add special behaviors.

As of this writing there are two defined directives, `@skip` and `@include`, which can apply to fields, fragment spreads and inline fragments:

```
query getUser($withAvatar: Boolean, $noFriends: Boolean){
  user {
    name
    avatar @include(if: $withAvatar)
    friends @skip(if: $noFriends) {
      name
    }
  }
}
```

The `@skip` and `@include` directives can be used to add customizability to queries (in terms of results) whilst still enabling query reuse, for purposes such as query caching.

In the future, new features could be added to the GraphQL spec by defining further directives, and some server implementations already define some experimentation ones.
