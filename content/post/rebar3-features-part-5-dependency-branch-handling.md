+++
author = "Tristan Sloughter"
categories = ["Erlang", "rebar3"]
date = 2015-09-21T19:24:21Z
description = ""
draft = false
slug = "rebar3-features-part-5-dependency-branch-handling"
tags = ["Erlang", "rebar3"]
title = "Rebar3 Features (part 5): Dependency Tracking"

+++

When starting rebar3 a goal was to never again end up in a situation where `rm -rf deps` to get the right set of dependencies was the quickest or only way forward.

An important piece of this is the new and improved [upgrade command](http://www.rebar3.org/docs/dependencies#upgrading-dependencies). However, this post will cover a less visible (it isn't a new command) improvement related to how rebar3 fetches and tracks dependencies each time the `compile` command is run.

Since there is no `get-deps` command, each time `compile` is run it relies on a task that verifies all dependencies are fetched and available in the appropriate build directory (specific to the profiles being used). Every resource type (`pkg`, `git`, `hg` and possibly [third party types](http://www.rebar3.org/docs/custom-dep-resources)) implements the behaviour `rebar_resource` which includes the function `needs_update/2`. If the application is found to exist already during the dependency verification phase and is in the lock file rebar3 then uses `needs_update/2` to check, unique to the resource type, if it is consistent with what is expected based on the lock file. If not, the source in the lock is fetched and replaces the current application.

An example project [dep_tracking](https://github.com/tsloughter/dep_tracking) has been created to show what this looks like when switching branches in a project that has differing dependencies between the branches. In this example the `master` branch depends on `cowboy` `1.0.3` and a branch `http2` depends on `cowboy` `master` where HTTP/2 support is being added. Just to show this doesn't change how transitive dependencies are handled, `ranch` has been added to the deps list as well, as a package `1.0.0` in `master` and `1.1.0` in `http2`.

Clone the repository and build as usual:

```
$ git clone https://github.com/tsloughter/dep_tracking
$ cd dep_tracking/
(master) $ cat rebar.lock
[{<<"cowboy">>,
  {git,"https://github.com/ninenines/cowboy.git",
       {ref,"b8e4115eb13488c517d8d8ef33c47d0eaa7838c6"}},
  0},
 {<<"cowlib">>,
  {git,"https://github.com/ninenines/cowlib.git",
       {ref,"d544a494af4dbc810fc9c15eaf5cc050cced1501"}},
  1},
 {<<"ranch">>,{pkg,<<"ranch">>,<<"1.0.0">>},0}].
(master) $ rebar3 compile
 ===> Verifying dependencies...
===> Fetching cowboy ({git,"https://github.com/ninenines/cowboy.git",{ref,"b8e4115eb13488c517d8d8ef33c47d0eaa7838c6"}})
===> Fetching ranch ({pkg,<<"ranch">>,<<"1.0.0">>})
===> Version cached at ~/.cache/rebar3/hex/default/packages/ranch-1.0.0.tar is up to date, reusing it
===> Fetching cowlib ({git,"https://github.com/ninenines/cowlib.git",{ref,"d544a494af4dbc810fc9c15eaf5cc050cced1501"}})
===> Compiling cowlib
===> Compiling ranch
===> Compiling cowboy
===> Compiling dep_tracking
```

Notice that in the case of a package (a [hex.pm](https://hex.pm) package to be specific) if a copy has already been fetched before, for any project, it is cached and will be reused instead of re-downloading, like in the case of git dependencies.

Now, even though all dependencies have been fetched and built, switching branches to the `http2` branch and running `compile` again will fetch the new versions of the dependencies that need updating and compile them:

```
(master) $ git checkout http2
(http2) $ cat rebar.lock
[{<<"cowboy">>,
  {git,"https://github.com/ninenines/cowboy.git",
       {ref,"0ffde50991502ed222005376a66791debc4b991c"}},
  0},
 {<<"cowlib">>,
  {git,"https://github.com/ninenines/cowlib.git",
       {ref,"14e597baa42a436469f99661ed7994685e269dc2"}},
  1},
 {<<"ranch">>,{pkg,<<"ranch">>,<<"1.1.0">>},0}].
(http2) $ rebar3 compile
===> Verifying dependencies...
===> Upgrading cowboy({git,"https://github.com/ninenines/cowboy.git",{ref,"0ffde50991502ed222005376a66791debc4b991c"}})
===> Upgrading ranch ({pkg,<<"ranch">>,<<"1.1.0">>})
===> Version cached at ~/.cache/rebar3/hex/default/packages/ranch-1.1.0.tar is up to date, reusing it
===> Upgrading cowlib ({git,"https://github.com/ninenines/cowlib.git",{ref,"14e597baa42a436469f99661ed7994685e269dc2"}})
===> Compiling cowlib
===> Compiling ranch
===> Compiling cowboy
===> Compiling dep_tracking
```

Rebar3 ensures the correct dependencies are fetched and built, simplifying working on different feature branches of your project or updating a branch with the upstream repository, without requiring any manual intervention. 

May `rm -rf deps` be forever expunged from the vocabulary of free developers!

