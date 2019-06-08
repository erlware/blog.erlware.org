+++
author = "Tristan Sloughter"
categories = ["buildpacks", "heroku", "node.js", "Opa", "web development"]
date = 2012-11-12T22:07:50Z
description = ""
draft = false
slug = "running-opa-applications-on-heroku"
tags = ["buildpacks", "heroku", "node.js", "Opa", "web development"]
title = "Running Opa Applications on Heroku"

+++

**TL;DR**  
  
As I've mentioned before, [Opa](http://opalang.org "Opa") is a new web framework that introduces not only the framework itself but a whole new language. A lot has changed in Opa since I last posted about it. Now Opa has a Javascript-esque look and runs on Node.js. But it still has the amazing typing system that makes Opa a joy to code in.  
  
The currently available [Heroku](http://heroku.com "Heroku") buildpack for Opa only supported the old, pre-Node, support. So I've created an all new buildpack and here I will show both a bit of how I created that buildpack and how to use it to run your Opa apps on Heroku.  
  
The first step was creating a tarball of Opa that would work on Heroku. For this I used the build tool [vulcan](https://github.com/heroku/vulcan "Vulcan"). Vulcan is able to build software on Heroku in order to assure what is built will work on Heroku through your buildpacks.  
<pre style="color:#dcdccc;background-color:#3f3f3f;">vulcan build -v -s ./opalang/ -c "mkdir /app/mlstate-opa &amp;&amp; yes '' | ./opa-1.0.7.x64.run" -p /app/mlstate-opa</pre>  
This command is telling vulcan to build what is in the directory _opalang_ with a command that creates the directory _/app/mlstate-opa_ and then runs the Opa provided install script to unpack the system. This is much simpler than building Opa from source, but it is still necessary to still use vulcan to create the tarball from the output of the install script to ensure paths are correct in the Opa generated scripts.  
  
After this run, by vulcan's default, we will have _/tmp/opalang.tgz_. I upload this to S3, so that our buildpack is able to retrieve it.  
  
Since Opa now relies on Node.js, the new buildpack must install both Node.js and the _opalang.tgz_ that was generated. To do this I simply copied from the [Node.js buildpack](https://github.com/heroku/heroku-buildpack-nodejs "Heroku Buildpack Node.js").  
  
If you look at the [Opa buildpack](https://github.com/tsloughter/heroku-buildpack-opa "Heroku Buildpack Opa") you'll see, as with any buildpack, it consists of three main scripts under _./bin/_: _compile_, _detect_ and _release_. There are three important parts for understanding how your Opa app must be changed to be supported by the buildpack.  
  
First, the _detect_ script relies on there being a _opa.conf_ to detect this being an Opa application. This for now is less important since we will be specifying the buildpack to use to the _heroku_ script. Second, in the _compile_ script we rely on there being a _Makefile_ in your application for building. There is no support for simply running _opa_ to compile the code in your tree at this time. Third, since Opa relies on Node.js and Node modules from _npm_ you must provide a _package.json_ file that the _compile_ script uses to install the necessary modules.  
  
To demostrate this I converted Opa's [hello_chat](https://github.com/MLstate/hello_chat "Opa hello_chat") example to work on Heroku, see it on Github [here](https://github.com/tsloughter/hello_chat "Heroku Opa hello_chat").  
  
There are two necessary changes. One, add the _Procfile_. A _Procfile_ define the processes required for your application and how to run them. For _hello_chat_ we have:  
<pre style="color:#dcdccc;background-color:#3f3f3f;">web: ./hello_chat.exe --http-port $PORT</pre>  
This tell Heroku that our web process is run from the binary _hello_chat.exe_. We must pass the _$PORT _variable to the Opa binary so that it binds to the proper port that Heroku expects it to be listening on to route our traffic.  
  
Lastly, a _package.json_ file is added so that our buildpack's compile script installs the necessary Node.js modules:  
<pre style="color:#dcdccc;background-color:#3f3f3f;">{  
  <span style="color:#cc9393;">"name"</span>: <span style="color:#cc9393;">"hello_chat"</span>,  
  <span style="color:#cc9393;">"version"</span>: <span style="color:#cc9393;">"0.0.1"</span>,  
  <span style="color:#cc9393;">"dependencies"</span>: {  
      <span style="color:#cc9393;">"mongodb"</span> : <span style="color:#cc9393;">"*"</span>,  
      <span style="color:#cc9393;">"formidable"</span> : <span style="color:#cc9393;">"*"</span>,  
      <span style="color:#cc9393;">"nodemailer"</span> : <span style="color:#cc9393;">"*"</span>,  
      <span style="color:#cc9393;">"simplesmtp"</span> : <span style="color:#cc9393;">"*"</span>,  
      <span style="color:#cc9393;">"imap"</span> : <span style="color:#cc9393;">"*"</span>  
  },  
  <span style="color:#cc9393;">"engines"</span>: {  
    <span style="color:#cc9393;">"node"</span>: <span style="color:#cc9393;">"0.8.7"</span>,  
    <span style="color:#cc9393;">"npm"</span>: <span style="color:#cc9393;">"1.1.x"</span>  
  }  
}</pre>  
With these additions to _hello_chat_ we are ready to create an Opa app on Heroku and push the code!  
<pre style="color:#dcdccc;background-color:#3f3f3f;">$ heroku create <span style="color:#7f9f7f;">--stack</span> cedar <span style="color:#7f9f7f;">--buildpack</span> https://github.com/tsloughter/heroku-buildpack-opa.git  
$ git push heroku master</pre>  
The output from the push will show Node.js and npm being install, followed by Opa being unpacked and finally make being run against hello_chat. The web process in _Procfile_ will then be run and the output will provide a link to go to our new application. I have the example running at [http://mighty-garden-9304.herokuapp.com](http://mighty-garden-9304.herokuapp.com)  
  
Next time I'll delve into database and other addon support in Heroku with Opa applications.

