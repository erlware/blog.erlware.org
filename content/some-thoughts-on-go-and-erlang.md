+++
author = "Tristan Sloughter"
type="post"
categories = ["Erlang", "fault tolerance", "Go"]
date = 2014-04-28T01:41:16Z
description = ""
draft = false
slug = "some-thoughts-on-go-and-erlang"
tags = ["Erlang", "fault tolerance", "Go"]
title = "Some Thoughts on Go and Erlang"

+++

**UPDATE:** I'm seeing that I did not make the point of this post clear. I am not saying Go is wrong or should change because it isn't like Erlang. What I am attempting to show is the choices Go made that make it not an alternative to Erlang for backends where availability and low latency for high numbers of concurrent requests is a requirement. And notice I'm not writing this about a language like Julia. I have heard Go pitched as an alternative to Erlang for not only new projects but replacing old. No one would say the same for Julia, but Go and Node.js are seen by some as friendlier alternatives. And no, Erlang isn't the solution for everything! But this is specifically about where Erlang is appropriate and Go is lacking.  
  
I'm going to attempt to leave out my subjective opinions for disliking parts of Go, such as syntax or lack of pattern matching, and explain objective reasons for the language and runtime not being fit for certain types of systems. But I'll start with the good.  
  
# Where Go Shines  
  
#### Clients  
  
As Rob Pike [wrote](http://commandcenter.blogspot.it/2012/06/less-is-exponentially-more.html "Less is exponentially more "), his biggest surprise was Go is mostly gaining developers from Python and Ruby, not C++. To me this trend has been great to see. No more slow clients installed through pip or gems! (Though for some reason Node.js for clients is growing, wtf [Keybase](https://keybase.io/ "Keybase")?)  
  
Go provides developers with a fast and easy to use high level, statically typed language with garbage collection and concurrency primitives. It would be great for C++ developers to move to Go as well, the programs that crash constantly on my machine are proprietary C++ that love to misuse memory -- Hipchat and Spotify. But Rob Pike pointed out that the C++ developers don't want the simplified, yet powerful, world of Go. Ruby and Python developers, rightly, do.  
  
#### Tooling  
  
Getting up and going with building executables depending on third party libraries can be done easily without depending on third party tools, it all comes with Go. While the tools aren't perfect, there are tools to fill in some gaps like [Godep](https://github.com/tools/godep), it is still a huge win for the language.  
  
# Where Go Falls Short  
  
Some of Go's design decisions are detrimental when it comes to writing low-latency fault-tolerant systems.  
  
#### Concurrency  
  
Yes, I listed concurrency primitives as a plus in the first section. It is in the case of replacing Ruby or Python or C++ for clients. But when it comes to complex backends that need to be fault-tolerant Go is as broken as any other language with **shared state**.  
  
#### Pre-emptive Scheduling  
  
Here Go has gotten much better. Go's pre-emptive scheduling was done on syscalls but now pre-emption is done when a goroutine checks the stack, which it does on every function call, and this may be marked to fail (causing pre-emption) if the goroutine has been running for longer than some time period. While this is an improvement it still lags behind Erlang's reduction counting and newly added dirty schedulers for improved integration with C.  
  
#### Garbage Collection  
  
In Go garbage collection is **global mark and sweep**. This pauses all goroutines during the sweep and this is terrible for latency. Again, low latency is hard, the more the runtime can do for you the better.  
  
#### Error Handling  
  
This isn't just about having no exceptions and the use of checking if a second return value is nil. Goroutines have **no identity** which means Go **lacks the ability to link or monitor goroutines**. No linking (instead using panic and defer) and no **process isolation** means you can not fall back on crashing and restarting in a stable state. There will be bugs in production and a lot of those bugs will be [Heisenbugs](http://citeseer.ist.psu.edu/viewdoc/download;jsessionid=285F73A4236CA1AE99EAB2439FFFB266?doi=10.1.1.59.6561&amp;rep=rep1&amp;type=pdf), so being able to layout processes, isolated from each other but linked based on their dependencies, is key for fault tolerance.  
  
And on top of these major omissions in dealing with faults Go has **nil**. How in 2014 this is considered OK I haven't wrapped my mind around yet. I'll just leave it at that, with a befuddled look.  
  
#### Introspection  
  
Not having a REPL is annoying for development, but no **remote shell** for running systems is a deal breaker. Erlang has impressive **tracing** capabilities and tools built on those capabilities like [recon_trace](http://ferd.github.io/recon/recon_trace.html "recon_trace"). Erlang's introspection greatly improves development as well as maintenance of complex running systems.  
  
#### Static Linking  
  
Yes, another thing that was also in the positives but becomes a negative when used in systems that are expected to be long running. While having no linking does means slower execution it gives Erlang advantages in the case of code replacement on running systems. It is important to note that due to Erlang's scheduling and garbage collecting strategies many of these speed tradeoffs do not mean Erlang will be slower than an implementation in another language, especially if the Erlang implementation is the only one still running.  
  
#### Code Organization  
  
The OTP framework provides libraries for common patterns. OTP not only means less code to write and better abstractions but also improves readability. Following the OTP standards with applications, supervisors and workers (gen_server, gen_fsm, gen_event) means a developer new to the program is able to work down through the tree of processes and how they interact. Go's channels, unidentifiable goroutines and lack of patterns to separate goroutines into separate modules leads to much harder code to reason about.   
  
# Can or Even Should Go Change?  
  
Erlang has been around for decades and Go is the new kid on the block, so can Go improve in these areas? Some, yes, but most of it can not because of the choices made in the design of the language which were not fault tolerance and low latency.  
  
This doesn't mean Go is "bad" or "wrong". It simply makes different choices and thus is better suited for different problems than a language like Erlang.

