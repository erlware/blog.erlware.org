+++
author = "Tristan Sloughter"
type="post"
categories = ["Erlang", "rebar3", "docker"]
date = 2018-09-18T06:09:11Z
description = ""
draft = false
slug = "rebar3-building-docker-images"
tags = ["Erlang", "rebar3", "docker"]
title = "Rebar3: Building Docker Images"

+++

### How I cut the time it takes to build an Erlang docker image in half.

While adding support for a rebar3 option to only build dependencies and not project applications, `rebar3 compile --deps_only`, I realized it might already actually work for the Docker use case I had in mind.

So I gave it a try and, yup, turns out it does!

The idea is to cache a docker layer of the built dependencies to reuse if they haven't changed. This is done by copying only the config files, which if changed will then invalidate the next Dockerfile commands, and build only the dependencies in the next layer. After that you copy in the source of your project as usual and build the release.

```
# build and cache dependencies
COPY rebar.config rebar.lock /usr/src/app/
RUN rebar3 compile

# copy in the vonnegut source and build release
COPY . /usr/src/app
RUN rebar3 as prod tar
```

No new functionality needed, just plain `rebar3 compile` does what we need when it is given a directory with only the config and/or lock file. After copying in the rest of the project `rebar3 as prod tar` will compile the project's source (the `tar` provider depends compile, so we don't need to call compile directly) and build a release tarball we can copy to a new image in the next stage.

A quick test with the [vonnegut](https://github.com/tsloughter/vonnegut/blob/7dd7fd0bcad3437a2ca231e066eae9710c9bce03/Dockerfile) Dockerfile I found the build times dropped from **~30** seconds to **~14** seconds. The initial build with only the Erlang image cached, so the packages and rebar3 still have to be installed, is around 50 seconds.

Note that this also works when only copying `rebar.lock`. Only copying the lock file may be beneficial to some who don't care about caching plugins and prefer to be able to change the rebar config without it invalidating the cache.

Also, we will still be merging in the `--deps_only` option as it likely has uses outside of building Docker images.

