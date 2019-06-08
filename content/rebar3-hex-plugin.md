+++
author = "Tristan Sloughter"
type="post"
categories = ["Erlang", "rebar3"]
date = 2015-10-05T19:50:56Z
description = ""
draft = false
slug = "rebar3-hex-plugin"
tags = ["Erlang", "rebar3"]
title = "Rebar3 Hex Plugin"

+++

No plugin is needed for using Hex packages in your project, covered in documentation [here](http://www.rebar3.org/docs/dependencies#package-manager-dependencies), but at this time the plugin [rebar3_hex](https://hex.pm/packages/rebar3_hex) is needed for publishing. The plugin provides support for registering with [hex.pm](http://hex.pm) and publishing packages. For instructions on registering see the [rebar3 hex.pm documentation](https://hex.pm/docs/rebar3_publish), here we'll cover publishing a package with some unique features to the rebar3 plugin.

### rebar3 hex cut

The `cut` command provides some automation on top of `publish`. The rebar3 hex plugin itself is a good example of a use case for this command. The `.app.src` file for the plugin looks like:

```
$ cat src/rebar3_hex.app.src 
{application, rebar3_hex,
  [{description, "Hex.pm plugin for rebar3"},
   {vsn, "git"},
   {applications, [kernel, stdlib, jsx]},
   {contributors, ["Tristan Sloughter"]},
   {licenses, ["MIT"]},
   {links, [{"Github", "https://github.com/tsloughter/rebar3_hex"}]}]}.

```

`contributors`, `licenses`, `links` are `hex` specific metadata. For the `vsn` of the application a commonly used feature of rebar3 is used where `"git"` is in the `.app.src` file and replaced at compile time with the latest git tag, if the current `HEAD` isn't on a specific tag the last tag with appended build metadata, as documented on [semver.org](http://semver.org/), is used.

Running `cut` will first ask for what increment of the version to perform, major, minor or patch. The version, based on the git tag, is then incremented and a new tag is created. After publishing the package `cut` asks if the git tag should be pushed to `origin`. 

In the case an application has a static version string (not one like `"git"`) `cut` will increment the version and update `.app.src` and create a `version bump` commit and push to `origin`. 

Below is the output from publishing a version of `rebar3_hex` with the `cut` command:

```
(master) $ rebar3 hex cut             
Select semver increment or other (Current 1.7.1):
1) patch
2) minor
3) major
4) other
[1-4] > 1
===> Creating new tag v1.7.2...
Publishing rebar3_hex 1.7.2
  Dependencies:
    jsx 2.6.1
  Excluded dependencies (not part of the Hex package):
    
  Included files:
    src/rebar3_hex.app.src
    src/rebar3_hex.erl
    ....
Proceed? ("Y")> Y
===> Published rebar3_hex 1.7.2
Push new tag to origin? ("Y")> Y
===> Pushing new tag v1.7.2...
```

### Publishing from an umbrella project

In an umbrella project there are multiple applications in the project. Since a package must be a single application the entire project can not be published as one. The rebar3 hex plugin will ask which application, or all, in the umbrella the user wants to publish. 

Remember to update the local rebar3 package registry with `rebar3 update` to fetch the latest published packages.

With [hex.pm](http://hex.pm) and [rebar3 hex](https://hex.pm/docs/rebar3_publish) an efficient and effective Erlang package ecosystem can grow.

