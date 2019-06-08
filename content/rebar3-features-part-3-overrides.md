+++
author = "Tristan Sloughter"
categories = ["Erlang", "rebar3"]
date = 2015-09-15T00:32:30Z
description = ""
draft = false
slug = "rebar3-features-part-3-overrides"
tags = ["Erlang", "rebar3"]
title = "Rebar3 features (part 3): Overrides"

+++

What do you do when a dependency has settings in its `rebar.config` that are causing you problems? Maybe it includes dependencies that are not needed in the general case, like `meck` or `edown`. Or it could have set a required OTP version that isn't accurate and you want to remove. Or the app could contain C code that needs compiling and had relied on rebar2's port compiler. These problems often lead to forks of projects, which isn't good for anyone, so in rebar3 we've added a feature called `overrides`.

Overrides allow any `rebar.config` at a higher level than a dependency to either replace or add to the configuration of all or an individual application at a lower level.

The type spec for overrides looks like:

```erlang
{overrides, [{add, atom(), [{atom(), any()]}
             | {override, atom(), [{atom(), any()]},
             | {override, [{atom(), any()]} 
            ]}.
```

The [bitcask](https://github.com/basho/bitcask/) application from Basho is configured to be built with rebar2, but instead of forking the project and patching the config, a user can instead simply add the below overrides section to their `rebar.config`:

```erlang
{overrides, 
  [
    {override, bitcask,
      [{deps, []},
       {plugins, [pc]},
       {artifacts, ["priv/bitcask.so"]},
       {provider_hooks, [{post, 
                          [{compile, {pc, compile}},
                           {clean, {pc, clean}}]
                         }]}
      ]
    }
  ]
}.
```

These overrides for `bitcask` replace its `deps` entry with an empty list, this removes the `meck` dependency which is only needed if you were running tests and the `cuttlefish` dependency which isn't required. Plus, since the port compiler functionality was removed in rebar3, the [port compiler](http://www.rebar3.org/docs/using-available-plugins#port-compiler) plugin must be added and hooked in to `compile` and `clean` with `provider_hooks`.

The rest of the guts of the override will be covered in future posts, see the docs for more information on [provider_hooks](http://www.rebar3.org/docs/configuration#section-provider-hooks), [artifacts](http://www.rebar3.org/docs/configuration#artifacts) and [plugins](http://www.rebar3.org/docs/using-available-plugins)

