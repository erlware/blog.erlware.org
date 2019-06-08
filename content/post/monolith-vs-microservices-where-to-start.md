+++
author = "Eric Merritt"
date = 2015-06-22T22:27:54Z
description = ""
draft = false
slug = "monolith-vs-microservices-where-to-start"
title = "Monolith vs Microservices: Where to start"

+++

There is a debate going around how to do the initial design of a new system in the context of Microservices. Should you start with a Monolithic approach initially and move to a Microservices later or use Microservices from the beginning?

#### Introduction

Martin Fowler recently wrote article called '[Monolith First](http://martinfowler.com/bliki/MonolithFirst.html)' that talks about how to get started on a new Microservice project. Stefan Tilkov wrote a response [Don't start with a monolith](href{http://martinfowler.com/articles/dont-start-monolith.html) arguing the reverse. As you would expect from the authors involved, both articles are well thought out and full of great information. 

Mr. Fowler posits that breaking things into services is error prone. You can't know ahead of time what the correct break out is so you should postpone that as late as possible. His implication is the main benifit of Microservices are their ability to scale. Mr. Tilkov hits on the point that the best time to architect a system is when you are starting out. Both authors, seem to miss the miss the point that these two options are not mutually exclusive.

#### The Modeling failure

These Engineers are thinking about Microservices as a unit of distribution rather then a unit of decomposition. Instead of thinking about Microservices as a way to structure their programs they should think of them as a modeling construct for those programs. Conflating Microservices-as-modeling-construct with Microservices-as-unit-of-distribution is like conflating [CORBA](https://en.wikipedia.org/wiki/Common_Object_Request_Broker_Architecture) with an Object System itself. It only makes sense in a very narrow context. 

I have the good fortune of having spent a significant part of my career writing systems in Erlang. [Erlang](http://www.erlang.org) is a distributed, fault tolerant language that uses processes as the fundamental unit of decomposition in the same way that Objects are the fundamental unit of decomposition in languages like Java and C++. The difference is that processes are Microservices. They are completely independent and have an explicit API and life cycle. In fact, I have been describing programming in Erlang, as modeling using very fine grained SOA, for almost fifteen years. I wish I had coined the term 'Microservices' instead, but the idea is the same. In Erlang There is no other approach to modeling. So you build your systems based on these tiny communicating services. It allows you to build your system as a group of small services that provide and receive work from one another. Erlang has another feature that makes all this possible. That feature is [Location Transparency](http://en.wikipedia.org/wiki/Location_transparency). Location Transparency means that I don't have to worry about where a service is to be able to communicate with it. I only need some name, a unique identifier. I can use the system facilities to communicate with the service identified by than name and have some high expectation that the service will receive it. 

With those two features Erlang has given me enough flexibility so that I can get all the benefits of composing my system in Micorservices and all the benefits of having the system be monolithic when it makes sense for it to be monolithic. Essentially, I don't have to worry about distribution during design and implementation at all. Erlang allows me to develop, test and do the initial deployments on a single node then, as the scale increases add new hardware to the system. This allows me to push out taking the additional conceptual load of distribution to as late a point as possible. 

#### Doing It Right in Other Languages

This isn't really a Microservice vs Monolith debate. Its a reaction to the lack of certain fundamental properties in existing Microservice platforms for other languages. One of the most important properties is [Location Transparency](http://en.wikipedia.org/wiki/Location_transparency). Often DNS or other location metadata is hard coded into the system, forcing the implementer to make decisions at compile time that constraint what is deployed at run time. Locational Transparency allows you to design your system using Microservices as a modeling construct without worrying about those services will be deployed.

In total, there are four properties that a Microservice framework needs to support. These are:


* Microservices are cheap to define.
* Microservices are Locationally Transparent.
* Service calls are context aware.
* Groups of services can be declaratively described for deployment.

A framework that fully supports thees for properties allows us to use Microservices as a unit of modeling in any language. Lets deep-dive into each property individually.

##### Micro services are cheap to define

Using Microservices must be as efficient to design and implement as possible. In the best case it's at least as efficient to implement as the native module constructs in a language, usually an Object or a Module. You also can't force the definition of a microservice to be in some foreign syntax like XML or Json. There will be a lot of Microservices and the creation of those services needs to flow with the language itself. So, in Java, we might take use attributes to take an object and expose its methods to a service compiler. This compiler could be run at compile time or at runtime. The details don't matter. In C++ we could use templates. In OCaml, the solution would be camlp4. All of these solutions use the same approach. They give the developer the ability to mark a Microservice as a Microservice and expose it's API without significant development overhead.

##### Microservices are Locationally Transparent

Microservices must be named in some unique way. Those names must be propagated throughout the system. That may be a single node on a developers desktop or ten thousand nodes spread through-out data centers in cities around the world. Systems that have this property allow names to be used as a composition and allows for useful Locational Transperancy. 

##### Service calls are context aware

Service calls are the ubiquitous form of work delivery in Microservice based systems. These calls must be as inexpensive as possible. The best way for to accomplish this goal is to make the call infrastructure context aware. When the Provider and the Consumer of the service are both on the same node, then the call must be made in the most efficient way possible. Preferably this would devolve into a simple function call. At the very least, serialization and deserialization should be avoided. With out this feature, the performance of a system that uses Microservices as a modeling approach will suffer. 


##### Declaratively describe groups of services as nodes

It is important that we can easily change the composition of a node. That we can, through some declarative means change which services run where in our system. This must also be done without recompiling the system. In the best case, it would also be done without redeploying the system, though in many cases that is to much to ask for. 

So there must be some configuration based, preferably declarative approach to describing nodes in the network that comprises the distributed system. 

##### Conclusion

The debate about how to design the initial architecture of a new system, Monolith vs Microservice, is predicated on the false assumption that these two approaches are mutually exclusive. That Microservices are a means of distribution rather than an architectural choice. By questioning that assumption and creating a Microservice framework that supports the key properties that we have talked about in this article, we can build flexible, scalable systems that are also easy to design test and build.

