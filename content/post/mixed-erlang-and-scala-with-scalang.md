+++
author = "Tristan Sloughter"
date = 2011-09-20T20:26:13Z
description = ""
draft = false
slug = "mixed-erlang-and-scala-with-scalang"
title = "Mixed Erlang and Scala with Scalang"

+++

This is a summary of a talk by [Cliff Moon @moonpolysoft](http://twitter.com/moonpolysoft) given at Strangeloop about building mixed Erlang and Scala systems with Scalang. Boundary does network analytics as a service. Their architecture uses a mixture of Erlang and Scala. Erlang is very very good at doing things like no down time deploys. We can have very low downtime on public facing parts of the system and we don't even have to go down for deploys. On the data processing side one of the things that erlang is very bad at is dealing with numbers, and generally anything where mutability has a high value.   
  
Trying to make the scala side talk do the Erlang side was required to handle the language choices for the system. Turns out that Erlang ships with Jinterface which is just the thing - or so it seem. Unfortunately it ended up being really really cumbersome. Jinterface is at the wrong level of abstraction. Erlang is all about actors and Jinterface only exposes mailboxes. All the rich interface you get in erlang with actors goes away when you are stuck with only mailboxes. The other problem is it is not performant. Primitives end up getting wrapped twice, first by Jinterface and then by a case class in scala which is just want to heavy weight when trying to process millions of pieces of data.   
  
They decided to take a step back, something that would be easier to use. They were looking for more correctness in behavior, things kind of behave like Erlang actors. They wanted performance, and then simplicity; not having to deal with custom serializers and other such cruft. The internal architecture is built on NIO sockets and Netty. There are also a bunch of codecs to do encoding and decoding between erlang and scala. There is also a delivery system wich deals with registration and actors which run in Jetlang - an actor framework for the JVM.  
  
The main interface into the system on the JVM side is something called a Node - this should be very familiar to Erlangers. It takes a node name and a magic cookie. So, pretty much exactly what you would expect.   
  
Once you have a node you want to make a process. Processes are spawned and messages are sent with the ! operator, just like in Erlang. You can send messages to a Pid, you can send messages to a local registered name, and you can send messages to a remote registered name by supplying a name node tuple. So basically just like Erlang.   
  
[![Cliff Moon talking about processes in scalang](http://erlware.files.wordpress.com/2011/09/scalang_processes.jpg "scalang_processes")](http://erlware.files.wordpress.com/2011/09/scalang_processes.jpg)  
  
### Error Handing in Scalang  
  
Scalang fires a link breakage exit signal anytime a Scalang process throws an uncaught exception. It works between Erlang and JVM. The one problem is that this is not preemptive on the JVM side as lightweight preemptive actors on the JVM side seems hard to do.   
  
### Erlang to Scala Type Mappings  
  
Most things are a one to one mapping for primitives. Anything that does not fit, like numbers, will be tured into something reasonable on the scala side. If just that is not quite good enough for you; you wnat to do rich type mappings. You can use a rich type mapping plugin to turn rich types into records and vice-versa.   
  
### Scalang Services  
  
One of the big things about Erlang is OTP. You typically use gen_servers, behaviors. These behaviors give you messaging primitives for sync and async and lots of other good stuff. Scalang wants to be able to interact with gen_servers transparently on the other side. So three functions are implemented for Scalang processes:  
  
handleCall  
handleCast  
handleInfo  
  
These will look very familiar for most Erlangers (if you are coding in OTP like you should be). Scalang also supports anonymous processes. You can just spawn processes with funs for those times when you don't want a gen_server.   
  
### Runtime Metrics  
  
This is what boundary does, so they wanted to bake them into all of their JVM stuff. Scalang has a full suite of runtime metrics. You get things like meters showing how many messages have come across the wire for each process. Histograms for process performance. Time spent in serialization. Message queue sizes and quite a number of other metrics. The idea was to make it similar to pulling up a remote shell into an erlang instance and being able to query to see where the bottlenecks are.   
  
### Scalang JVM Performance Tuning  
  
We are all about running fast here. Scalang aims to make things easily tunable. It turns out one of the best way to performance tune is to screw around with the thread pools. The ThreadPoolFactory lets you screw around with different implementation. There are 4 kinds  
  
Boss Pool - initial connection and accept handling  
Worker Pool - non blocking reads and writes  
Actor Pool - process callbacks   
Batch executor - per process execution logic  
  
Editorial: The system really looks to be quite powerful. It allows for the features I describe above as well as easy remote shell invocation of JVM nodes. It actually interacts nicely with EPMD for native feeling messaging. The system seems to be abstracted more appropriately than any Erlang to X intercoms library I have run across. I look forward to hearing about experiences using it.   
  
Martin Logan ([@martinjlogan](http://twitter.com/martinjlogan)) also, if you are into distributed systems and metrics you should check out [Camp DevOps Conf in Chicago this Oct](http://campdevops.com)  
  
Here is where you can find the code and example usage information: [https://github.com/boundary/scalang](https://github.com/boundary/scalang)

