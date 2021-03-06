+++
author = "Tristan Sloughter"
type="post"
categories = ["Cowboy", "Erlang", "OTP", "rebar", "sinan"]
date = 2012-05-05T01:33:07Z
description = ""
draft = false
slug = "sinan-releases-and-being-right"
tags = ["Cowboy", "Erlang", "OTP", "rebar", "sinan"]
title = "Sinan Releases and Being Right"

+++

[Fred](http://ferd.ca/), of [Learn You Some Erlang for Great Good](http://learnyousomeerlang.com/ "Learn You Some Erlang"), today posted on his blog about the problems around how rebar handles releases, [Rebar Releases and Being Wrong](http://ferd.ca/rebar-releases-and-being-wrong.html "Rebar Releases and Being Wrong"). The problems he mentions and a few others are why, despite giving it a legitimate shot, I have found rebar unusable for my workflow to be efficient and stable while adhering to OTP standards at the same time.  
  
I suggest first reading his post, if you already use rebar, and then continuing on with the rest of this.  
  
I'll start with an example on the generation of a project containing two applications and a dependency from one of those applications of [cowboy](https://github.com/extend/cowboy "Cowboy"). Next, I'll create a release (and in the process a deployable target system) to show the difference in how [sinan](https://github.com/erlware/sinan "Sinan") handles this process.  
  
TL;DR [Sinan](https://github.com/erlware/sinan "Sinan") does OTP the right way, rebar does not.  
  
First, you can download the latest version sinan from this [link](https://github.com/downloads/erlware/sinan/sinan "Sinan Download"), it is simply an executable escript, so '_chmod +x sinan_' and put it in your **PATH** and you are good to go.  
  
Sinan provides a 'gen' command to create your project. I include the output of the steps I took to build this project. Sinan assumes this is a multiple application project, but if you give "y" instead it will create a directory structure similar to rebars default structure with a src/ directory instead of a lib/ directory.  
<pre style="color:#dcdccc;background-color:#3f3f3f;">$ sinan gen  
Please specify your name   
your name&gt; Tristan Sloughter  
Please specify your email address   
your email&gt; tristan@mashape.com  
Please specify the copyright holder   
copyright holder ("Tristan Sloughter")&gt;   
Please specify name of your project  
project name&gt; rel_example  
Please specify version of your project  
project version&gt; 0.0.1  
Please specify the ERTS version ("5.9")&gt;   
Is this a single application project ("n")&gt;   
Please specify the names of the OTP apps that will be developed under this project. One application to a line. Finish with a blank line.  
app&gt; app_1  
app ("")&gt; app_2  
app ("")&gt;   
Would you like a build config? ("y")&gt; y  
Project was created, you should be good to go!</pre>  
We now have a project named rel_example and can see the generated contents.  
<pre style="color:#dcdccc;background-color:#3f3f3f;">$ cd rel_example/  
$ ls  
config  doc  lib  sinan.config</pre>  
Before going further I add the line {include_erts, true}. to sinan.config so that a generated tarball of the release contains erts and can be booted on a machine without Erlang installed.  
<pre style="color:#dcdccc;background-color:#3f3f3f;"><span style="color:#bfebbf;">$ </span>cat sinan.config  
{project_name, rel_example}.  
{project_vsn, <span style="color:#cc9393;">"0.0.1"</span>}.  
  
{build_dir,  <span style="color:#cc9393;">"_build"</span>}.  
  
{ignore_dirs, [<span style="color:#cc9393;">"_"</span>, <span style="color:#cc9393;">"."</span>]}.  
  
{ignore_apps, []}.  
  
{include_erts, true}.</pre>  
A tree structure view of the generated project is below:  
<pre style="color:#dcdccc;background-color:#3f3f3f;">.  
├── config  
│   └── sys.config  
├── doc  
├── lib  
│   ├── app_1  
│   │   ├── doc  
│   │   ├── ebin  
│   │   │   └── overview.edoc  
│   │   ├── include  
│   │   └── src  
│   │       ├── app_1_app.erl  
│   │       ├── app_1.app.src  
│   │       └── app_1_sup.erl  
│   └── app_2  
│       ├── doc  
│       ├── ebin  
│       │   └── overview.edoc  
│       ├── include  
│       └── src  
│           ├── app_2_app.erl  
│           ├── app_2.app.src  
│           └── app_2_sup.erl  
└── sinan.config</pre>  
You'll see we have a lib directory with two applications containing their source files under a src directory. Now in order to boot the release we'll create, we need to remove a couple tings from each supervisor. Instead of creating something for them to supervise just remove the variable AChild and replace [AChild] with [].  
  
Next, so we have a third party dependency in the example, add cowboy to the applications in nano lib/app_1/src/app_1.app.src:  
<pre style="color:#dcdccc;background-color:#3f3f3f;">{applications, [kernel, stdlib, cowboy]},</pre>  
Sinan provides a depends command to show the depenedencies of the project and where they are located:  
<pre style="color:#dcdccc;background-color:#3f3f3f;">$ sinan depends <span style="color:#7f9f7f;">-v</span>  
<span style="color:#cc9393;">starting: depends</span>  
Using the following lib directories to show resolved dependencies and where it found them:  
  
    /home/tristan/.kerl/installs/r15b/lib  
    /home/tristan/Devel/rel_example/_build/rel_example/lib  
  
compile time dependencies:  
  
runtime dependencies:  
  
    kernel                    2.15       : /home/tristan/.kerl/installs/r15b/lib/kernel-2.15  
    stdlib                    1.18       : /home/tristan/.kerl/installs/r15b/lib/stdlib-1.18  
    cowboy                    0.5.0      : /home/tristan/.kerl/installs/r15b/lib/cowboy-0.5.0  
  
project applications:  
  
    app_1                     0.1.0      : /home/tristan/Devel/rel_example/_build/rel_example/lib/app_1-0.1.0  
    app_2                     0.1.0      : /home/tristan/Devel/rel_example/_build/rel_example/lib/app_2-0.1.0</pre>  
Now lets build a release and target system.  
<pre style="color:#dcdccc;background-color:#3f3f3f;">$ sinan dist</pre>  
After running the dist command we have a _build directory that we find the following structure. I removed the files/dirs under each app to shorten the listing.  
<pre style="color:#dcdccc;background-color:#3f3f3f;">_build/  
├── rel_example  
│   ├── bin  
│   │   ├── rel_example  
│   │   └── rel_example-0.0.1  
│   ├── erts-5.9  
│   │   ├──   
│   ├── lib  
│   │   ├── app_1-0.1.0  
│   │   │   ├──   
│   │   ├── app_2-0.1.0  
│   │   │   ├──   
│   │   ├── cowboy-0.5.0  
│   │   │   ├──   
│   │   ├── kernel-2.15  
│   │   │   ├──   
│   │   └── stdlib-1.18  
│   │       ├──   
│   └── releases  
│       └── 0.0.1  
│           ├── rel_example.boot  
│           ├── rel_example.rel  
│           ├── rel_example.script  
│           └── sys.config  
└── tar  
    └── rel_example-0.0.1.tar.<span style="color:#e0cf9f;font-weight:bold;text-decoration:underline;">gz</span></pre>  
Sinan has created a lib directory containing all necessary applications for our release as well as the needed files for booting the release. Additionally the dist command creates a tar.gz for easy deployment. But if we simply want to run our release where we are we can:  
<pre style="color:#dcdccc;background-color:#3f3f3f;">$ _build/rel_example/bin/rel_example  
Erlang R15B (erts-5.9)  
  
<span style="color:#8cd0d3;">[64-bit]</span> <span style="color:#f0dfaf;font-weight:bold;">[smp:4:4]</span> <span style="color:#8cd0d3;">[async-threads:0]</span> <span style="color:#f0dfaf;font-weight:bold;">[hipe]</span> <span style="color:#8cd0d3;">[kernel-poll:false]</span>  
  
<span style="color:#bc8383;font-weight:bold;text-decoration:underline;">Eshell</span> V5.9  (abort with ^G)  
1&gt;</pre>  
  
This is only the tip of the iceberg of what sinan is capable of. I can't go into all of it here but I'll mention that you are able to define multiple releases for a project to generate and which of your project apps to include in each. Additionally you are able to provide a custom rel file if you require tweaks.  
  
The important part to take away from this post is the structure of what you are working with when using sinan and how it is based on OTP standards, both for the source you work on and the results of the build process under _build/.

