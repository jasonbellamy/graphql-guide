---
title: Schemas
order: 1
---

The starting point to any GraphQL stack is defining the schema. The GraphQL schema describes what the shape of the data requirements of queries against your server look like.

<h2 id="schema-language">GraphQL Schema Language</h2>

There are many [GraphQL server libraries](servers.html) available, in many different programming languages, and the way you construct your schema will depend on the library you are using. However, there is a also a semi-official shorthand ["schema language"](https://raw.githubusercontent.com/sogko/graphql-shorthand-notation-cheat-sheet/master/graphql-shorthand-notation-cheat-sheet.png) which, although it doesn't support all GraphQL schema features, provides a language agnostic way to describe schemas.

Typically you would not write you application's schema in the schema language (although the reference JavaScript implementation does allow you to, and it can be useful for prototyping); however its is useful for our examples to use it. Translating it to your server implementation's schema constructors should be reasonably mechanical.

<h2 id="basics">Schema Basics</h2>

A GraphQL schema is in essence the definition of a type system--how the data available to you client is related and what queries can be made to retrieve it. Additionally it describes the mutations available (and the data they can return).

A schema is constructed of a set of [objects](#objects) types--which are maps of named [fields](#fields), each of which is another type, either another object, or a [scalar](#scalar) base type. You can think of the object types as the nodes in the type system's "graph" and the scalar types as leaves (technically nodes of degree one).

<h3 id="operations">Operations</h3>

The entry points to a schema (the fields of the `schema` object) are the three kinds of operation available:

 - All schemas must have `query` object type whose fields are the set of queries endpoints of your application.

 - A schema may optionally have `mutation` object type, whose fields are the set of mutations available.

 - Additionally the schema may have a `subscription` object type, whose fields are the set of subscriptions.

A schema with at least one of all three kinds of operation would look like:

```
schema {
  query: Query
  mutation: Mutation
  subscription: Subscription
}
```

In this example the three object types `Query`, `Mutation` and `Subscription` would be defined elsewhere. The names of those types can be anything, but it's sensible to pick the names above.

The *fields* of those types define the actual entrypoints or instances of that operation that are available. Each GraphQL document (combination of queries, mutations and subscriptions) must include at least one of those operation entrypoints. So, we could define the `Query` type above as:

```
type Query {
  me: User
  feed: [Item]
  user(id: ID): User
}
```

In the above example, the schema provides three fields on the query type (or "starting queries"), and any GraphQL query operation must start with one of those three entry points. From there, we can build an arbitrary data query, based on the fields that the types consist of (ie. using fields of `User` and `Item` and then their subtypes).

<h3 id="objects">Objects</h3>

The top-level "operation" types and the majority of other types in a typical schema are of the object type. These types consist of a named set of other types (called the "fields" of this type), which can themselves be object types, or simple scalar types.

For example, we could define the `User` type above:

```
type User {
  name: String
  avatar: String
  friends: [User]
}
```

This means that any query that reaches a `User` can select any combination of the three fields above in constructing a data query.

<h3 id="fields">Fields</h3>

The fields on an object type consist of a name and another type. Additionally, fields can take *arguments*, to enable a parameterized result. For instance in our definition of the `Query` type above:

```
type Query {
  me: User
  feed: [Item]
  user(id: ID): User
}
```

We include a `user` field which is parameterized by the `id` argument of type `ID`.

> Although it is typical that the fields of the query operation's type are parameterized (after all, they are queries), it's important to remember that *any* field on *any* object type can be parameterized.
> For instance, the `avatar` field on the `User` type above could have been parameterized to take a `size: Int` argument.

<h3 id="scalars">Scalars</h3>

<h3 id="input-objects">Input Objects</h3>

<h2 id="advanced">Advanced Schemas</h2>

<h3 id="enums">Enumerations</h3>

<h3 id="interfaces">Interfaces</h3>

<h3 id="unions">Unions</h3>
