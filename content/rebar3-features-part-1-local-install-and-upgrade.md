+++
author = "Tristan Sloughter"
type="post"
categories = ["Erlang", "rebar3"]
date = 2015-09-11T04:21:52Z
description = ""
draft = false
slug = "rebar3-features-part-1-local-install-and-upgrade"
tags = ["Erlang", "rebar3"]
title = "Rebar3 Features (part 1): Local install and upgrade"

+++

**Update January 22, 2016:** The original post has been modified to reflect that the commands `install` and `upgrade` were moved from the `unstable` namespace to the `local` namespace. Also added is use of the environment variable `$REBAR3_ERL_ARGS` in the run script which allows the user to specify custom Erlang VM arguments like: `REBAR3_ERL_ARGS="+A10" rebar3 shell`.

[Rebar3](https://rebar3.org) comes with a lot of new and improved features. I'll be publishing posts here to highlight some of these features over the coming weeks.

#### Installing and Upgrading Rebar3

Rebar is an escript bundle, and this has been very important to its ease of use, mainly because they let Erlang users have a single file acting as an executable, regardless of the underlying Erlang installation, which can even be committed to a project's repository.

So, rebar3 is also an escript. Pre-built escripts can be downloaded from s3: 

```
$ wget https://s3.amazonaws.com/rebar3/rebar3
```

or clone and bootstrap the git repo:

```
$ git clone https://github.com/rebar/rebar3.git
$ cd rebar3
$ ./bootstrap
```

However, escripts do have their drawbacks. They are slower to start, rely on the old Erlang io server, making `rebar3 shell` not act exactly the same as an `erl` shell, and it can lead to repos storing the escript along with the code but never upgrading it.

So in rebar3 we've introduced the ability to extract the escript archive along with a run script to `~/.cache/rebar3/`, plus a command that will fetch and do the same for the latest escript release of rebar3 from s3.

The `install` command will be under a provider namespace, originally it lived under `unstable`, the namespace for experimental features that are likely to change in the near future and that aren't yet considered stable, but now is in the `local` namespace:

```
$ ./rebar3 local install
===> Extracting rebar3 libs to $HOME/.cache/rebar3/lib...
===> Writing rebar3 run script $HOME/.cache/rebar3/bin/rebar3...
===> Add to $PATH for use: export PATH=$HOME/.cache/rebar3/bin:$PATH
```
Follow the instructions for adding the rebar3 bin directory to your `$PATH`, and optionally add it to your shell's configuration, such as `~/.bashrc` or `~/.zshrc`.

To upgrade rebar3 use `rebar3 unstable upgrade` which will fetch the latest escript and extract it:

```
$ rebar3 local upgrade
```

We hope this new method for installing, upgrading and running rebar3 will allow people to be comfortable with not bundling rebar3 in their project's repositories, resolve any compilation speed differences between rebar3 and alternative Erlang build tools and help keep everyone up to date with the latest features and bug fixes.

