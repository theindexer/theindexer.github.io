---
layout: post
title:  "Domain Driven Design?"
date:   2020-05-27 22:32:14 -0400
categories: patterns opinion
---
# Domain Driven Design

## What is?

Domain Driven Design is a bunch of buzzwords stuck together.
For that reason, it probably means a lot more things than it was intended to mean.
I'm writing about my interpretation of it, as it pertains to me and my work. I'm sure if I got slash-dotted which I assume doesn't happen anymore or showed up on hackernews, I'd get lots of comments about how what I'm talking about isn't Domain Driven Design. I'm okay with that.

## Me

I write Java for a post-startup b2b microservice hotshot unicorn rockstar company or something.

## How I practice

### Service Encapsulation

My non-technical operating definition of a service is something that _services_ my requests. I have a lot of requests. If I'm managing a user, I want to know their profile information. I want to know their subscription information. I want to know their last login date. Depending on my architecture, these could all be stored in the same place, or vastly different places. If I'm not thinking about services at all, I find the most direct line of access to that information and use it. If it's in a database, I read the database. If it's in S3, I grab me some credentials and hop on over.

In a service-oriented world, I ask the profile service to give me the profile information. I ask the subscription service for the subscription information. How I get the last login date is an exercise left to the reader. In java, I'll have something like `SubscriptionService` and `RpcSubscriptionService` which I'm not entirely satisfied with but works. An equivalent could be `ISubscriptionService` and `SubscriptionServiceImpl` or whatever programmers of yore did. I'm not a fan of that, YMMV. Point is, we have an interface and an implementation.

Sometimes I get asked something like "Do we really need an interface for this?" The answer is no, not really. Not for the code to work, at least. 
There's a 1% chance that we're going to change the implementation of this class and we can just eat the expected value of not having the interface cluttering up the codebase, if that was all we cared about. I'm of the mind that we should create it anyway, because it makes us think.
Without an interface, you don't need an implementating class. Without an implementating class, you're free to have your clients and database connections and whatever else fly around your codebase, sprinkling information wherever your app needs it. This sounds pretty awesome, until you try to test your code. Or a new argument gets added to the client. Or a database column changes its definition.
Then you're stuck playing a fun game of `grep` (well, `rg`, or `ag` or `git grep` or literally anything but `grep`), combing your code for references to the offending call and updating it / wrapping it in a backwards compatible way / unleashing the unholy hells of `PowerMock` upon it.

A service is an API. It forms a contract between your code and some other code (possibly also yours) to deliver the return types from your inputs. An API is simple to reason about. You can look it up and say "In my application, we only call the subscription service to show the tier the user is in", because you only have one method in your service called `retrieveTier(int userId)`. Someone on the subscriptions team can grep your code and figure out that no one is using the `1.5BillingDatesAgo` field and deprecate it, because product isn't going to let them outright remove it anyway.

At work, our microservices typically communicate over rpc through clients. Typically, each server has one client implementation. This is a service API. However, this is not the form of encapsulation I'm looking for. The first problem is that many services may use this client, and not all of them use every method on the client. The second is that the results from client calls often contain information we're not concerned with. This serves a nice segue into the next point.

### Domain Models

This is more contentious than Service Encapsulation, as far as conversations with coworkers has led me to believe. A domain model is how my system views something that came from another system. Typically, my services take in domain models as arguments and return domain models. For an rpc service, this means that the transport layer is pretty much always immediately converted away from. I frequently use protobufs to define rpc communication. As soon as the protobuf comes in to my service, I exctract the fields I need into a domain model and return that. Sometimes, the domain model looks exactly the same as the protobuf. Sometimes, the protobuf looks exactly the same as the domain model on the other end. As our code lives in a monorepo, I've heard that we should just share the same model on both sides of the client/serve relationship, and while we're at it use the protobuf as that model. I don't buy it, but I'll admit I'm still tempted from time to time.

### Separation of Concerns

Server code usually talks to the database. Rpc code is serialized to go across the wire, with an emphasis on backwards compatability and evolvability. Client code does whatever I need it do. At the moment, it may be the case that the same object serves all of these needs. Will that always be the case? Well, we have some dumb-as-brick objects for which it probably holds true. But we definitely have plenty of objects that look and act different. If you want to arbitrate when you need separate models on a case-by-case basis, be my guest. But in my short-lived experience, a blanket policy works better.

## Further Thoughts

This hasn't even touched dependency management, so that should probably be the next one. I'd also like to write about each these in more detail, with examples.
