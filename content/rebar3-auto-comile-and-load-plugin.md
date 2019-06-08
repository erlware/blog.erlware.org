+++
author = "Tristan Sloughter"
categories = ["Erlang", "rebar3"]
date = 2015-09-26T21:57:18Z
description = ""
draft = false
slug = "rebar3-auto-comile-and-load-plugin"
tags = ["Erlang", "rebar3"]
title = "Rebar3 Auto Compile and Load Plugin"

+++

During development the Erlang shell is often used for quickly testing functionality. Erlang's ability to reload modules while running makes this workflow even more efficient. To go a step further in removing manual intervention tools like [sync]() and [active]() have been created. These libs will listen for file modifications, recompile and reload the changed modules. 

With [rebar3](http://rebar3.org) there is a plugin [rebar3_auto](https://hex.pm/packages/rebar3_auto) which will start the shell, begin listening for modifications in the source directories of the project and recompile/reload when changes occur. Add to your global rebar3 configuration file to be able to use it on any of your projects `~/.config/rebar3/rebar.config`:

```
{plugins, [rebar3_auto]}.
```

Running `rebar3 auto` will start the shell the same as running rebar3 shell but will be listening for file changes in your project's application source directories. When a file is change it will message the rebar3 agent to run compile and reload modules.

```
(master) $ rebar3 auto                       
===> Fetching rebar3_auto ({pkg,<<"rebar3_auto">>,<<"0.2.0">>})
===> Downloaded package, caching at /home/tristan/.cache/rebar3/hex/default/packages/rebar3_auto-0.2.0.tar
===> Fetching enotify ({pkg,<<"enotify">>,<<"0.1.0">>})
===> Version cached at ~/.cache/rebar3/hex/default/packages/enotify-0.1.0.tar is up to date, reusing it
===> Compiling enotify
===> Compiling rebar3_auto
Setting up watches.  Beware: since -r was given, this may take a while!
Watches established.
Erlang/OTP 18 [erts-7.1] [source] [64-bit] [smp:4:4] [async-threads:0] [hipe] [kernel-poll:false]

Eshell V7.1  (abort with ^G)
1> rlx_util:test().
** exception error: undefined function rlx_util:test/0
```
Oops, our function doesn't exist. Switch to your editor and add to `rlx_util.erl` a function `test/0` and export it, when you save you'll see it is recompiled and now available in the shell: 

```
2> 
Verifying dependencies...
Compiling relx

2> rlx_util:test().
hello
```

In the shell all the features of the `shell` command, its configuration and the rebar3 agent are available, see the [docs](http://www.rebar3.org/docs/commands#shell) for more information.

