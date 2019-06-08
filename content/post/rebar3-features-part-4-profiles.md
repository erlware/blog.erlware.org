+++
author = "Tristan Sloughter"
categories = ["Erlang", "rebar3"]
date = 2015-09-21T02:04:20Z
description = ""
draft = false
slug = "rebar3-features-part-4-profiles"
tags = ["Erlang", "rebar3"]
title = "Rebar3 Features (part 4): Profiles"

+++

Running tests and need `meck` or `proper`? Building docs and want `edown`? Bundling up a target system and want to include `erts` and turn off relx's `dev_mode`? Rebar3 now has you covered for these scenarios through `profiles`. 

Profiles can be named with any atom, can add new items to the configuration, or prepend new options to existing elements, and multiple profiles can be combined themselves.

The two special profiles are `default` and `global`. `default` is the profile everything is run under, with output going to `_build/default/`, unless another is specified in addition to `default`. When multiple profiles are used together the output directory is the profiles concatenated together with `+`, for example running `rebar3 as test,prod <task>` would produce `_build/test+prod/`, however the actual combination of profiles used in that run is `default,test,prod`, in the output `default` is always removed from the beginning unless it is the only profile in use.

The other special case for how profiles decide where output is written is `global` always refers to `~/.cache/rebar3/`. 

Providers are able to set profiles they will run under (in addition to `default`) with the `{profiles, [atom()]}` option to `providers:create/1`. Four providers that come with `rebar3` specify a profile: `eunit`, `ct` and `cover` use `test` and `edoc` uses `docs`.

Examples of profile usage can help give an idea of how you might use them in your projects.

Since `eunit`, `ct` and `cover` run with the `test` profile adding `deps` specific to tests, like `meck` and `eunit_formatters`, which will be used when running `rebar3 ct` or any of the others, there is no need to include `as test` in the run, but to be clear, profiles are deduplicated so `rebar3 as test ct` will still be `_build/test` and *not* `_build/test+test`.

```
{profiles, 
  [{test, [{deps,
            [
             meck,
             {eunit_formatters, {git, "git://github.com/seancribbs/eunit_formatters", {branch, "master"}}}
            ]},

            {eunit_opts, [
              no_tty,
              {report, {eunit_progress, [colored, profile]}}
             ]}
           ]
   },
....
```
Another common dependency that in rebar2 would be included in the main dependency list and thus be fetched even when using used as a dependency is `edown`. With the `docs` profile that `edoc` runs with that is solved by moving `edown` under the profile:

```
{profiles, [{docs, 
            [{deps, [
              {edown, {git, "git://github.com/uwiger/edown.git", {branch, "master"}}}
             ]}]
           }]
}.
```

When developing a release, it is useful to use relx's `dev_mode` and to set `include_erts` to false. But when building a release for production you'll want the opposite. In this case, unlike with tests and docs, it is required to specify the profile you want to run the command with. Running `rebar3 release` will run as default, so with `dev_mode` true and `include_erts` false, while `rebar3 as prod release` pulls in the settings from the `prod` profile making `dev_mode` false and `include_erts` true.

```
{relx, [...
        {dev_mode, true},
        {include_erts, false},
        ...
       ]}.

{profiles, 
  [{prod, [{relx, [
                   {dev_mode, false},
                   {include_erts, true}
                  ]
          }]
   },
```

The `global` profile is used in particular for plugins that the user defines in their personal `rebar.config`. For example I run `rebar3 as global plugins upgrade` to upgrade the two plugins in my `~/.config/rebar3/rebar.config`:

```
{plugins, [rebar3_hex, rebar3_run]}.
```

Profiles are an important addition to the rebar configuration for making development and dependency management simpler. Please be a good Erlang citizen and separate your dependencies into the appropriate profiles, those who depend on your application will appreciate it.

