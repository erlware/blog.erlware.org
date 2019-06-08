+++
author = "Tristan Sloughter"
date = 2013-06-04T19:41:53Z
description = ""
draft = false
slug = "universal-makefile-for-erlang-projects-that-use-rebar"
title = "Universal Makefile for Erlang Projects That Use Rebar"

+++

[![](http://www.erlangcamp.com/assets/erlangcamp-logo-afdc8efebef6893f692323ad60d06871.png)](http://www.erlangcamp.org)  
  
_This post sponsored by [ErlangCamp 2013](http://www.erlangcamp.org) in Nashville and Amsterdam_  
  
At this point in the game nearly every Erlang project uses [Rebar](https://github.com/rebar/rebar). The problem with that is that Rebar's approach to the command line and command dependency chaining is leaves a lot to be desired. You tend to end up typing the same command with the same options list over and over again during the course of your work. Because of the poor dependency chaining you often must retype the same sequence of commands as well. Finally, there are certain things (like [Dialyzer](http://www.erlang.org/doc/man/dialyzer.html) support) that Rebar does not support.  
  
In our Erlware projects, we want a consistent and recognizable entry point into the build process. For that reason we tend to treat Rebar as a low level tool and drive it and the other build tools I mentioned with a Makefile. That makes it far easier for us, as developers, to chain rules as needed and create additional rules that add features to the build system. This allows us to integrate other tools seamlessly into the build experience. At Erlware, we have developed a pretty standard Makefile that can be used with little or no changes from project to project. You can find the whole of that Makefile [here](https://gist.github.com/ericbmerritt/5706091). However, I will work my way through a few parts of it explaining so you understand what is going on and can make changes relevant to your project.  
  
The main targets this Makefile supports are as follows:  
  
*   deps: Pull the project dependencies (called automatically as needed)  
*   update-reps: Update the dependencies (never called automatically)  
*   compile: Compiles the project  
*   doc: Builds the edoc documentation  
*   test: Compiles the code and runs the tests (designed to be called by a human)  
*   dialyzer: Build the dependency PLT and run dialyzer on the project  
*   typer: Run [Typer](http://www.erlang.se/workshop/2005/TypEr_Erlang05.pdf‎) on the project  
*   shell: Bring up an Erlang shell with all the dependencies already loaded and unit tests compiled and available.  
*   pdf: Turn your README.md into a pdf using pandoc (pretty useful at times, but completely optional)  
*   clean: Delete the build output files  
*   distclean: Remove the build output files as well as the project PLT file and all the dependencies  
*   rebuild: Do a dist clean, rebuild everything from scratch and run both the tests and dialyzer  
Now that we have an idea of the targets available lets work through the major points of the Makefile.  
  
**Defining Variables**  
  
<pre>  
ERLFLAGS= -pa $(CURDIR)/.eunit -pa $(CURDIR)/ebin -pa $(CURDIR)/deps/*/ebin  
  
DEPS_PLT=$(CURDIR)/.deps_plt  
DEPS=erts kernel stdlib  
</pre>  
  
At the top of the make file a few variables that are set. For the most part you don't ever have to touch any of these with the exception of DEPS. The DEPS variable provides a list of dependent applications that are used by Dialyzer to build the dependency PLT file. The others are ERLFLAGS, which is used by the shell command to correctly make your code available in the shell, and DEPS_PLT, which points to the location where the project PLT file will be located.  
  
**PLT Files and Dialyzer**  
  
<pre>  
$(DEPS_PLT):  
	@echo Building local plt at $(DEPS_PLT)  
	@echo  
	dialyzer --output_plt $(DEPS_PLT) --build_plt \  
	   --apps $(DEPS) -r deps  
  
dialyzer: $(DEPS_PLT)  
	dialyzer --fullpath --plt $(DEPS_PLT) -Wrace_conditions -r ./ebin  
</pre>  
  
This is how the Dialyzer command is run. The main things to notice here are that a PLT file specific to the project is built using the dependencies that you described at the top of the file in the DEPS variable. Building a per project PLT solves a raft of potential problems but has the downside that the first run of Dialyzer or the first run after a rebuild can take several minutes as it analyzes all of the dependencies to build the PLT file.  
  
**Rebuilding**  
  
Rebuilding is basically a completely clean rebuild and test of the system. You should run this code before you submit a PR or share code with your peers. It basically tries to ensure that you have not forgotten or left off anything that is needed.  
  
**Conclusion**  
  
You can, quite literally, drop this makefile into your project and use it today with only some very minor modification to the DEPS variable. If you are not already using something like this in your project I encourage you to add this Makefile now. It will save you a lot of tedious typing and make your build process much clearer to your users.  
  
**Alternatives**  
  
There are a few alternatives to this approach out there. These are quite good if somewhat more complex.  
  
*   Make setup that does not use rebar - [https://github.com/extend/erlang.mk](https://github.com/extend/erlang.mk)  
*   A more complete, but more complicated Makefile generator - [https://github.com/opscode/concrete](https://github.com/opscode/concrete)

