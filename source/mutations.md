---
title: Mutations
order: 3
---

You use queries to fetch data in GraphQL; when you want to modify data you use a **mutation**.

In the first instance, a mutation is simply a named RPC (remote procedure call), basically equivalent to a data modification API in many other systems such as REST.

Where things get interesting in GraphQL is considering the *return value* of a mutation. In most applications, a mutating API will return a minimal amount of data to the client, which--if the client needs to find out what's changed as a result of the mutation, which it typically does--will require a second request to fetch the changed data.

In some cases, mutating APIs are expanded to return a set of data to the caller; however it's not generally possible to know exactly what data any caller will need, which may mean the API endpoint returns *too much* data. We can see exactly the same over/underfetching problem that [motivated](index.html#motivation) GraphQL in the first place--and the solution is the same: to allow the caller to exactly the data needed after the mutation is done.

With that in mind, a mutation ends up looking almost exactly the same as a query:

```
mutation updateAvatar($avatar: String) {
  updateAvatar(avatar: $avatar) {
    name
    avatar(size: 30)
  }
}
```

This mutation will return something like:

```
{
  "data": {
    "updateAvatar": {
      "name": "Tom Coleman",
      "avatar": "https://some.cdn/avatar_30.png"
    }
  }
}
```

The big difference between the above and a similar-shaped query is in the semantics: a query should be *idempotent* and *side-effect-free* (i.e. you can run it many times without changing anything), whereas a mutation is very much not.

<h2 id="defining-mutations">Defining Mutations</h2>

Mutations are defined in a schema in a very similar way to queries; you simply attach them as fields of the the `Mutation` [operation type](schemas.html#operations). In the schema language, you might write something like:

```
type Mutation {
  updateAvatar(avatar: String!): Profile
}

schema {
  mutation: Mutation
}
```

What this says is there is an `updateAvatar` mutation available, it takes a non-null `String` as an argument, and can return fields from a `Profile` user.

The type of the mutation field is an important consideration of the definition. When you are modifying or inserting a single object (such as the profile here), it makes sense to root return queries at that type; if the client wants to fetch data related to the change (such as perhaps comments made by that profile), you can add fields to that type to associate the data.

If the mutation is more "global" in effect, and can affect *any* data in the application, the mutation could return the base query type (so the mutation caller can run any query they like). Such mutation would look like:

```
type Query {
  me: Profile
  feed: [Activity]
}

type Mutation {
  upgradeAccount($level: String): Query
}

schema {
  query: Query
  mutation: Mutation
}
```

<h3 id="serving-mutations">Serving Mutations</h3>

Mutations are typically transported over the same [network layer](queries.html#network-layer) as queries, and use the same underlying protocol (such as HTTP).

Also server implementations typically *resolve* mutations in the [same way as queries](queries.html#query-resolution). Again, the main difference is in semantics; unlike other resolvers, resolvers of a mutation are *expected* to modify data, rather than just query for it.

Like any other resolver, a mutation resolver should return the data pertaining to the type that it returns: in the case where the mutation is mapped to the root query type, it would return nothing at all.
