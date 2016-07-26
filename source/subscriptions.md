---
title: Subscriptions
order: 4
---

We've seen how to use queries to fetch a once-off snapshot of the state of a set of data from our server. What if we want to keep up to date as that dataset changes?

One approach is to poll the server and keep running requests over a time interval. This is a valid approach, but with the advent of newer technologies and techniques that allow pushing data bidirectionally--and thus allowing the server to "push" data to the client, it also makes sense to support "subscribing" to a changing dataset.

GraphQL subscriptions support whilst offering the same "fetch exactly what you need" approach of GraphQL in general.

<h2 id="basics">Subscription Basics</h2>

A subscription is an operation, specified by the `subscription` keyword. A subscription looks, and in many ways, acts identically to a query:

```
subscription userSubscription {
  user {
    name
    avatar
    friends {
      name
    }
  }
}
```

The difference is that although the subscription returns an initial dataset that looks much--or perhaps exactly--like the results of the equivalent query:

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

The subscription also will stream changes to that result set as the underlying data changes.

<h2 id="defining">Defining Schemas</h2>

Similarly to [mutations](mutations.html) and [queries](queries.html), you define subscriptions on the `Subscription` [operation type](schemas.html#operations):

```
type Subscription {
  feed: [Activity]
}

schema {
  subscription: Subscription
}
```


Often, (depending on your subscription infrastructure), you may just want to enable the exact same set of query entrypoints for your subscriptions as for your regular query, in which case, you can just mirror the `Query` operation type:

```
type Query {
  feed: [Activity]
}

type Subscription {
  feed: [Activity]
}

schema {
  query: Query
  subscription: Subscription
}
```

Typically, you'll want a separate type for the subscription because the action of the resolvers will be quite different.

<h2 id="serving">Serving Subscriptions</h2>

As they rely on "server-push" technologies, such as websockets, and return a stream of data, rather than a single dataset, the mechanisms for serving subscriptions are significantly different.

<h3 id="network-layer">Network Layer</h3>

Subscriptions need to be sent over some "subscription-aware" network layer protocol, such as [DDP](https://www.meteor.com/ddp) over a websocket. This allows the server to send updates or new documents as the queries results change, and the client to stop the subscription when it no longer wishes to receive results.

<h3 id="resolving">Resolving Subscriptions</h3>

Your subscription resolvers are usually quite different from query resolvers because they must return a stream of data rather than a single dataset. Depending on the [server technology](servers.html) you use, this can mean returning something like an observable, calling callbacks, or the like.

The mechanism for actually tracking changes to your dataset and patching them into your subscription resolver is out of scope of GraphQL, and is totally up to you and dependent on the technology you are using.
