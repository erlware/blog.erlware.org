+++
author = "Tristan Sloughter"
type="post"
categories = ["Erlang", "rebar3"]
date = 2015-10-03T22:15:36Z
description = ""
draft = false
slug = "rebar3-features-part-6-_checkouts-2"
tags = ["Erlang", "rebar3"]
title = "Rebar3 Features (part 6): _checkouts"

+++

In a build tool there is often a balancing act between ensuring repeatability and efficiency for the user. Wanting to make modifications on a dependency of your project is a common case of this. In rebar2 you could simply modify the source under `deps/` and running `rebar compile` would pick those up. This meant that the contents of `deps/` are not representative of the dependencies listed in `rebar.config`.

With rebar3 a dependency is never rebuilt, even if a source file changes. Before compiling changes to the project apps' source files rebar3 verifies that all dependencies are valid (meaning all `artifacts` exist and the `.app` file of the dependency matches the existing `beam` files for the dependency) and any that are not are fetch and/or built. To facilitate the workflow where a developer wants to modify one of these dependencies rebar3 introduces a feature call "checkout dependencies".

Checkout dependencies work by the user creating a directory `_checkouts` at the top level of the rebar3 project. Under `_checkouts/` a copy of or symlink to an application's directory can be placed to have that copy take precedence over any dep in a config file, whether it has already been fetched and built or not. The apps under `_checkouts` are treated like project apps and any changes made to source files will be compiled. There is no need to delete any copy of the applications under `_build/`, rebar3 handles properly setting the code paths when building applications so only the version found under `_checkouts` will be used. 

The checkouts also work for plugins. In the example below the structure of a `_checkouts` directory with two applications is shown. One being `erlware_commons` and the other a plugin `rebar3_auto`. This means that running `rebar3 auto` from this project will use the `_checkouts` copy of `rebar3_auto` and not one fetched to `_build/default/plugins`, a great way for performing manual testing on plugins you are developing.

```
$ tree _checkouts                 
_checkouts
├── erlware_commons -> ~/code/erlware_commons
└── rebar3_auto -> ~/code/rebar3_auto
```

Because the dependency used by the project is now not what is fetched and locked based on `rebar.config` or `rebar.lock` they will be removed from `rebar.lock` the next time `compile` is run. This makes clear to the developer that what is being built locally is not what would be built if pushed and fetched by another user, breaking repeatability. The developer must acknowledge this by making a point to commit the new lock file, if for whatever reason that is indeed a dependency to be removed.

