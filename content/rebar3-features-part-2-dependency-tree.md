+++
author = "Tristan Sloughter"
type="post"
categories = ["Erlang", "rebar3"]
date = 2015-09-12T21:56:08Z
description = ""
draft = false
slug = "rebar3-features-part-2-dependency-tree"
tags = ["Erlang", "rebar3"]
title = "Rebar3 Features (part 2): Dependency Tree"

+++

`rebar3 tree` is a new command to allow the user to view what dependency pulled in each transitive dependency. This is especially useful with rebar3's dependency resolution strategy of "first wins".

Thanks for pushing for this feature goes to [Heinz N. Gies](https://github.com/Licenser/) and inspiration comes from [leiningen's](https://github.com/technomancy/leiningen) command `lein deps :tree`.

For an example I've cloned [chef-server](https://github.com/chef/chef-server/) and built `oc_erchef` under `src/oc_erchef`. It is unique because it has both a top level app `oc_erchef` under `src/` as well as additional project apps under `apps/`. 

Additionally, I've added `_checkouts/erlware_commons` to show how [checkout dependencies](http://www.rebar3.org/docs/dependencies#checkout-dependencies), an application linked to under `_checkouts/`, is moved to the top level and marked as `(checkout app)` and switched `lager` to a [hex](http://hex.pm) [package dependency](http://www.rebar3.org/docs/dependencies#package-manager-dependencies), the rest being [git sourced dependencies](http://www.rebar3.org/docs/dependencies#source-dependencies).

```
$ rebar3 tree
├─ chef_db─12.1.2-6047c67 (project app)
├─ chef_index─12.1.2-6047c67 (project app)
├─ chef_objects─12.1.2-6047c67 (project app)
├─ chef_test─12.1.2-6047c67 (project app)
├─ depsolver─12.1.2-6047c67 (project app)
├─ erlware_commons─0.16.0 (checkout app)
├─ oc_chef_authz─12.1.2-6047c67 (project app)
├─ oc_chef_wm─12.1.2-6047c67 (project app)
└─ oc_erchef─12.1.2-6047c67 (project app)
   ├─ bcrypt─0.0.0+build.87.ref085eb59 (git repo)
   ├─ chef_authn─0.0.0+build.86.refe7850d0 (git repo)
   ├─ darklaunch─0.0.0+build.72.ref05881cb (git repo)
   │  └─ meck─0.8.3 (git repo)
   ├─ efast_xs─0.1.0 (git repo)
   ├─ ej─0.0.0+build.87.ref132a9a3 (git repo)
   ├─ envy─0.0.0+build.38.ref954c87a (git repo)
   ├─ eper─0.90.0 (git repo)
   ├─ folsom─0.0.0+build.335.ref38e2cce (git repo)
   │  └─ bear─0.0.0+build.32.ref1192345 (git repo)
   ├─ folsom_graphite─0.0.0+build.41.refd4ce9bf (git repo)
   ├─ gen_bunny─0.1 (git repo)
   │  ├─ amqp_client─0.0.0 (git repo)
   │  └─ rabbit_common─0.0.0 (git repo)
   │     └─ gen_server2─1.0.0 (git repo)
   ├─ ibrowse─4.0.1.1 (git repo)
   ├─ jiffy─0.0.0+build.131.reff661ee9 (git repo)
   ├─ lager─2.1.1 (hex package)
   │  └─ goldrush─0.1.6 (hex package)
   ├─ mini_s3─0.0.1 (git repo)
   ├─ mixer─0.1.1 (git repo)
   ├─ neotoma─0.0.0+build.125.ref760928e (git repo)
   ├─ opscoderl_folsom─0.0.1 (git repo)
   ├─ opscoderl_httpc─0.0.1 (git repo)
   ├─ opscoderl_wm─0.0.1 (git repo)
   │  └─ webmachine─0.0.0+build.526.ref7677c24 (git repo)
   │     └─ mochiweb─2.9.0 (git repo)
   ├─ pooler─0.0.0+build.159.ref7bb8ab8 (git repo)
   ├─ sqerl─1.0.0 (git repo)
   │  └─ epgsql─3.1.0 (git repo)
   ├─ stats_hero─0.0.0+build.73.refff00041 (git repo)
   │  └─ edown─0.2.4+build.66.ref30a9f78 (git repo)
   ├─ sync─0.1.3 (git repo)
   └─ uuid─1.3.2 (git repo)
      └─ quickrand─1.3.2 (git repo)
```

