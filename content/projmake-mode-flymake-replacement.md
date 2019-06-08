+++
author = "Tristan Sloughter"
categories = ["Emacs", "Erlang", "Functional Programming", "OTP", "rebar"]
date = 2012-09-27T23:13:47Z
description = ""
draft = false
slug = "projmake-mode-flymake-replacement"
tags = ["Emacs", "Erlang", "Functional Programming", "OTP", "rebar"]
title = "Projmake-mode: Flymake Replacement"

+++

There is a great new [Emacs](http://www.gnu.org/software/emacs/ "Emacs") plugin from [Eric Merritt](http://blog.ericbmerritt.com/ "Eric Merritt") that like [FlyMake](http://www.emacswiki.org/emacs/FlyMake "FlyMake") builds your code and highlights within Emacs any errors or warnings, but unlike FlyMake builds across the whole project. You can clone the mode from here [projmake-mode](https://github.com/ericbmerritt/projmake-mode "projmake-mode")  
  
After cloning the repo to your desired location add this bit to your dot emacs file, replacing _&lt;PATH&gt;_ with the path to where you cloned the repo.  
  
[gist]3794732[/gist]  
  
This Emacs code also uses _add-hook_ to set _projmake-mode_ to start for _erlang-mode_ is loaded. Projmake by default knows how to handle [rebar](https://github.com/basho/rebar "rebar") and Make based builds so there is no setup after this, assuming your project is built this way.  
  
Here is my _Makefile_ for building Erlang projects with rebar, replace _PROJECT<em> with the name of your project:_</em>  
  
[gist]3795007[/gist]  
  
Now you can load Emacs and a file from your project and if it is an Erlang file due to the _add-hook_ function in our dot emacs file it will automatically load projmake-mode. You can add hooks for other modes or simply run _M-x projmake-mode_.  
  
For more documentation and how to extend to other types of projects check out the [documentation](https://github.com/ericbmerritt/projmake-mode/blob/master/README.md "README").

