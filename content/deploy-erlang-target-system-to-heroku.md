+++
author = "Tristan Sloughter"
date = 2014-08-28T02:54:05Z
description = ""
draft = false
slug = "deploy-erlang-target-system-to-heroku"
title = "Deploy Erlang Target System to Heroku"

+++

In this post these new tools will be used:  
  
*   [rebar3](https://github.com/rebar/rebar3 "Rebar3")
*   [hk](https://github.com/heroku/hk "Heroku Client") - new cli client for Heroku  
*   [hk slug plugin](https://github.com/tsloughter/hk-slug "hk slug plugin") - plugin for hk that uploads the target system  
*   [Heroku Cedar-14](https://blog.heroku.com/archives/2014/8/19/cedar-14-public-beta "Heroku Cedar 14") - new Heroku Cedar on Ubuntu 14.04  
*   [Heroku Slug API](https://devcenter.heroku.com/articles/platform-api-deploying-slugs "Heroku Slug API")  
  
First, clone _minasan_ and create the Heroku application on Cedar-14:  
<pre style="line-height:1.3em;font-family:monospace;color:#f8f8f2;background-color:#272822;">$ git clone https://github.com/tsloughter/minasan.git  
$ cd minasan  
$ heroku create --stack cedar-14</pre>  
  
Now that Heroku has the cedar-14 stack if you are also running a recent Linux distro you can upload the target system created by _relx_ directly to your app, before now we would have to build it on Heroku or in a system with an older glibc to work on Heroku's Ubuntu 10.04.  
  
Since _minasan_ is using binary packages and a fork of _rebar_ be sure to use the _rebar_ included in the repo, same goes for _relx _so that including the Procfile in the tarball works. The first step will be to update the package index for _rebar_, then compiling and building the release tarball (with _erts_ included and _dev-mode_ off so the Erlang runtime is included):  
<pre style="line-height:1.3em;font-family:monospace;color:#f8f8f2;background-color:#272822;">$ ./rebar3 update  
$ ./rebar3 compile
$ ./rebar3 do release -i true --dev-mode false, tar</pre>  
  
Using the new Slug API endpoint through _hk slug_ the tarball can be pushed directly as a slug to your app and then scale up the web process to at least 1:  
<pre style="line-height:1.3em;font-family:monospace;color:#f8f8f2;background-color:#272822;">$ hk slug _rel/minasan/minasan-0.0.1.tar.gz  
$ hk scale web=1  
$ hk open  
</pre>  
  
Your browser should now open to your new app.  
  
A few things to note:  
  
*   _'./rebar3 pkgs'_ will show you a list of available packages to use in _rebar deps_  
*   Currently '_hk slug'_ only support sending a tarball that does not yet have the structure of a slug. So it is unpacked and repacked. I plan to support directories and properly formatted tarballs.

