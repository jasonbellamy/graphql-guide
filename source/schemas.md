---
title: Schemas
order: 1
---

The starting point to any GraphQL stack is defining the schema. The GraphQL schema describes what the shape of the data requirements of queries against your server look like.

<h2 id="schema-language">GraphQL Schema Language</h2>

There are many [GraphQL server libraries](servers.html) available, in many different programming languages, and the way you construct your schema will depend on the library you are using. However, there is a also a semi-official shorthand ["schema language"](https://raw.githubusercontent.com/sogko/graphql-shorthand-notation-cheat-sheet/master/graphql-shorthand-notation-cheat-sheet.png) which, although it doesn't support all GraphQL schema features, provides a language agnostic way to describe schemas.

Typically you would not write you application's schema in the schema language (although the reference JavaScript implementation does allow you to, and it can be useful for prototyping); it is useful for our examples to use it. Translating it to your server implementation's schema constructors should be reasonably mechanical.

<h2 id="basics">Schema Basics</h2>

A GraphQL schema is in essence the definition of a type system--how the data available to you client is related and what queries can be made to retrieve it. Additionally it describes the mutations available (and the data they can return).

A schema is constructed of a set of [object](#objects) types--which are maps of named [fields](#fields), each of which is another type, either another object, or a [scalar](#scalar) base type. You can think of the object types as the nodes in the type system's "graph" and the scalar types as leaves (technically nodes of degree one).

A GraphQL schema is directly analogous to (and in many implementations actually is) a set of classes in a object-oriented programming language. Each class (or "object type") has a set of properties and methods (or "fields") that may be (or return) simple scalars or other class instances. Unsurprisingly, the analogy extends, and other object-oriented concepts, such as [type interfaces](#interfaces) and [union types](#unions) apply.

When thinking about a schema, it's useful to remember what a GraphQL query does: it _starts_ at a given [operation](#operations), and walks a (tree-like) path through the type system, choosing one or more fields from each object type that it encounters.

<h3 id="operations">Operations</h3>

The entry points to a schema (the fields of the `schema` object) are the three kinds of operation available:

 - All schemas must have `query` object type whose fields are the set of query endpoints of your application.

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

Scalars types are where the actual data of the type system lives. Although you can introduce your own scalar types into your schema, GraphQL ships with a collection of default scalar types:

 - `Int` - representing an Integer
 - `Float` - representing a Floating point number
 - `String` - A string of characters
 - `Boolean` - A `true`/`false` value
 - `ID` - An identifier -- a string that is used for data identification

When you declare a field of an object type to be a scalar type, it becomes an "end point" for a GraphQL query.

<h3 id="non-nulls">Non-null types</h3>

By default, all types in GraphQL can be represented by `null`, to indicate a missing value or error fetching a specific piece of data. In particular when an object type returns `null`, no data will be returned for sub-fields.

You can declare specific fields to be a "non-null type", which is a wrapper around another type that indicates that a `null` value is *not* allowed.

In the schema language, you can represent a non-null type by appending `!` after it:

```
# a person must have an id and name, but the avatar is optional
type Person {
  _id: ID!
  name: String!
  avatar: String
}
```

<h3 id="arrays">Array types</h3>

Typically, it's useful to have fields which are lists of objects or scalars. To do so, you can create another wrapper type, an array type. You can represent an array type in the schema language by surrounding an existing type with `[]`.

```
# a person has a list of persons for the `friends` field:
type Person {
  friends: [Person]
}
```

<h3 id="input-objects">Input Arguments and Objects</h3>

When you create a parameterized field, in simpler cases, the argument(s) to the field will be basic scalar or [enum types](#enums). You can also define default values for those types in your schema. In the schema language you can use `= value` to do so:

```
type Person {
  avatar(size: Int = 100): String
}
```

In some cases, the arguments will be complex enough to justify creating an input object type, which allows complex arguments that are validated by the type system.

An input object type is just like an object type, except it may not use [interfaces](#interfaces), [unions](#unions) or circular references to the same input object type.

For example, if we wanted to jazz up our `avatar` field on `Person`, we could take a variety of arguments:

```
input AvatarOptions {
  format: String
  filter: String
}

type Person {
  avatar(size: Int = 100, options: AvatarOptions): String
}
```

<h2 id="advanced">Advanced Schemas</h2>

With the above basics you can get started building schemas, but there are some extra features that will help you get the most out of your GraphQL schemas.

<h3 id="enums">Enumerations</h3>

When you have a scalar type that takes it value from a set list of values, it make sense to use an enumerated type. This often makes the most sense for an input argument. To continue our avatar example:

```
enum ImageFormat {
  JPG
  PNG
  GIF
}

input AvatarOptions {
  format: ImageFormat
}
```

It's not required, but by convention, you should use uppercase names for the enum values.

<h3 id="interfaces">Interfaces</h3>

You can achieve polymorphism in your type system by using an interface. You can set a field on an object type to be an interface, which is an *abstract* type that is fulfilled by one or more other object types. It more or less corresponds exactly the concept in a object-oriented programming language.

For example, in schema language:

```
interface FeedItem {
  title: String
}

interface Location {
  latitude: String
  longitude: String
}

type Photo implements FeedItem {
  title: String
  url: String
}

type Event implements FeedItem, Location {
  title: String
  latitude: String
  longitude: String
}

type User {
  lastActivity : FeedItem
}
```

<h3 id="unions">Unions</h3>

A union type is simply an option between one or more other types. This means that a field on an object can return one of several options depending on the specific object returned.

```
type Person {
  firstName: String
  lastName: String
}

type Company {
  name: String
  location: Location
}

union Contact = Person | Company

type AddressBook {
  contacts: [Contact]
}
```
