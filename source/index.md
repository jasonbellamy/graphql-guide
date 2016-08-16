---
title: Introduction
order: 0
---

Welcome to Apollo's GraphQL Guide. This guide provides an introduction to [GraphQL](http://graphql.org)--the next generation app data query language.

<h2 id="what-is-graphql">What is GraphQL?</h2>

GraphQL was designed (and has been actively used) by Facebook in 2012 to provide a common data interface for their mobile and web applications. GraphQL is optimized for the type of data that is typically needed by such applications; data that is hierarchical (a "graph") and which is queried directly from a client application.

A GraphQL query looks like:

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

And a response might look like:

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

<h2 id="motivation">Motivation</h2>

Any client application, that requires data from a server must fetch that data via some interface. Ideally the client would only fetch the data that the user of the app currently needs (such as what's currently visible on the screen).

Traditionally, a client app connects to a set of *server endpoints*, each of which provides a some set of data to the client. One popular scheme for organizing such endpoints is [REST](https://en.wikipedia.org/wiki/Representational_state_transfer).

In the best case, there is an endpoint that provides exactly the currently required data; in reality that's often not the case and the app ends up over-fetching data, or having to make multiple requests to the server (or both!). As multiple applications (such as a distinct web and mobile interfaces) start needing overlapping data, and as data requirements change across versions of a single application, providing tailored endpoints becomes more and more difficult.

Instead, GraphQL provides an approach where the client application requests exactly the data it needs, in the form of a *query*. The GraphQL server's *schema* defines the queries that are allowed and the shape of the data that are capable of returning, in a composable way---the client can then construct queries that give exactly the data that they need.

<h2 id="use-cases">Use Cases</h2>

From a client perspective, GraphQL is well suited to applications where the data requirements are varied and intersecting. This is often the case when you have:

 - Multiple clients built on the same data--for instance a web and mobile version of the same core application--but potentially also distinct applications that access the same underlying information.
 - Multiple versions of the same application running in the wild--for instance releases of native mobile applications whose data needs change over time.
 - A rapidly developed application where data APIs can be slow to change--for instance when the client team ships at a different schedule to the server team.

From a server perspective, GraphQL's data abstraction and query validation allows you write vastly simpler query endpoints that can combine multiple services in a straightforward yet flexible way. This helps if:

 - You are fetching data from multiple services or databases for each client request and combining them in a single response.
 - Your endpoints are complicated and brittle because their use is overloaded by multiple clients.
 - Analyzing the impact on your data services of a given client (or part of a client) is difficult due to the fact that many clients access the same endpoint.

<h2 id="how-it-works">How it works</h2>

At it's core, GraphQL is a specification of a query language. Given a *schema* (which is a set of named queries and the types of data they return), GraphQL specifies what a *query* against that schema can look like, when it is valid, and what the shape of the resulting data should look like. It also provides a mechanism for querying the schema itself to introspect information about the API. This is a powerful feature that allows for self documenting APIs and powerful developer tools.

<h3 id="how-it-works-on-the-server">On the Server</h3>

On the server, you define a GraphQL [schema](schemas.html) which outlines the set of queries that are available and the types of data they return. As the types can reference each other recursively, it is typically possible to construct arbitrarily complex queries on top of these endpoints.

For instance, the query [given above](#what-is-graphql) would be valid for a schema of the form:

```
type Person {
  name: String
  avatar: String
  friends: [Person]
}

type Query {
  user: Person
}

schema {
  query: Query
}
```

You can read more about schemas in our article on [the subject](schemas.html).

Alongside the schema, which speaks to what queries are valid, you also define [resolvers](resolvers.html), which are the way in which queries are turned into results.

A resolver is in essence a way to turn a materialize a field on a GraphQL type. So, a resolver for the above schema might detail how to get the list of `friends` for a given `Person`. As such a resolver is the point where your GraphQL server talks to the backing data store, be it a database or another service.

You can read more about resolvers in our article on [the subject](resolvers.html).

<h3 id="how-it-works-on-the client">On the Client</h3>

On the client, you construct GraphQL [queries](queries.html) that are sent to the GraphQL server. Typically you co-locate those queries with the user interface components for whom they are supplying data. This is a very different way to do it compared to a query defined in a server-side data endpoint!

Although you could simply send the query yourself over HTTP to a GraphQL server, and manually deal with the results, on the client you typically use a GraphQL client library, such as [Apollo Client](http://docs.apollostack.com/client) or [Relay](https://facebook.github.io/relay/) to manage your queries and cache their results. Due to the structured nature of GraphQL data and the introspection  allowed by GraphQL servers, GraphQL clients are able to perform many complex optimizations that wouldn't otherwise be possible.

<h2 id="what-is-apollo">What is Apollo?</h2>

Apollo is a set of GraphQL technologies and services designed to make GraphQL powerful and easy to use across many platforms. It includes a dedicated JavaScript client library ([Apollo Client](/client)) alongside integrations with client-side libraries, native mobile clients and utilities to improve the reference JavaScript server implementation ([Apollo Server](/server)).

Apollo also encompasses services to help you get the most out of your GraphQL stack. An initial product, [Apollo Optics](http://www.apollostack.com/optics) is currently in development.
