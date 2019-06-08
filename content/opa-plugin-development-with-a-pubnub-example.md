+++
author = "Tristan Sloughter"
categories = ["cloud", "Functional Programming", "Javascript", "Opa", "Web"]
date = 2011-11-24T04:03:07Z
description = ""
draft = false
slug = "opa-plugin-development-with-a-pubnub-example"
tags = ["cloud", "Functional Programming", "Javascript", "Opa", "Web"]
title = "Opa Plugin Development with a PubNub Example"

+++

This will be a two part series of posts on writing plugins for [Opa](http://opalang.org "Opa"). One of Opa's greatest features is you write everything in the functional, staticly typed Opa language. This even includes the frontend code you'd usually do in Javascript. This means your code is less error-prone which significantly reduces the amount of time spent on debugging code.  
  
But how do you use Javascript library X within your Opa code?  
  
While you can include Javascript code in a resource, like we do here for Google Analytics:  
<pre style="color:#00ff00;background-color:#000000;"><span style="color:#00ffff;">do</span> <span style="color:#87cefa;">Resource.register_external_js</span>(<span style="color:#ffa07a;">"/resources/js/google_analytics.js"</span>)</pre>  
This isn't what we want to do if we want to call external Javascript functions from Opa code. For this we create an Opa plugin. In this post I'll be showing how I built the Opa client side bindings for the [PubNub](http://www.pubnub.com/ "PubNub") Javascript API, [fork it on github](https://github.com/tsloughter/opa_pubnub_plugin "Opa PubNub Plugin") and see all the [other language bindings](https://github.com/pubnub/pubnub-api "PubNub API"). PubNub is a cloud hosted real-time publish and subscribe interface for all types of applications.  
  
Opa allows binding to external functions through its Binding System Library (BSL). In order to use this to create a plugin for some external Javascript, we create a file pubnub.js that we will compile to a plugin using _opa-plugin-builder_. Below is an example of one of the external functions that needs to be defined to use PubNub's publish.  
<pre style="color:#00ff00;background-color:#000000;"><span style="color:#ff0000;">##</span>register <span style="color:#eedd82;">publish :</span> string, string -&gt; <span style="color:#7fffd4;">void</span>  
<span style="color:#ff0000;">##</span>args(c, m)  
{  
    PUBNUB.publish({  
                       <span style="color:#eedd82;">channel</span> : c, <span style="color:#eedd82;">message</span> : m  
                   });  
  
    <span style="color:#00ffff;">return</span> js_void;  
}</pre>  
The first line registers the function _publish_ and specifies its arguments to be two strings and its return value to be _void_. The next line names the arguments. You can think of _##args_ like Javascript's _function_ keyword. In the function, we can use the PUBNUB library just like we would normally. Here we pass to publish the channel and the message to be published. Note that the message can be any Javascript object, even though its a string to the plugin. We simply must serialize the JSON before passing to _publish_, which we'll see shortly.  
  
Now after defining external functions for all PubNub api calls in the [pubnub.js](https://github.com/tsloughter/opa_pubnub_plugin/blob/master/src/plugin/pubnub.js "Pubnub.js") file we can compile it:  
<pre style="color:#00ff00;background-color:#000000;">$ opa-plugin-builder src/plugin/pubnub.js -o pubnub</pre>  
This creates the plugin pubnub.opp.  
  
To make using the plugin easier, we create a package in pubnub.opa that will contain a module _PubNub_. Below is the contents of pubnub.opa for the publish function that will show how to register the external functions and create a module named PubNub around them.  
<pre style="color:#00ff00;background-color:#000000;"><span style="color:#00ffff;">package</span> pubnub  
  
<span style="color:#b0c4de;">pubnub_publish =</span> <span style="color:#ffa07a;">%%pubnub.publish%%</span>  
  
<span style="color:#cd3700;">@client</span> <span style="color:#b0c4de;">publish_client =</span> pubnub_publish  
  
<span style="color:#b0c4de;">PubNub =</span> <span style="color:#00ffff;">\{\{</span>  
  <span style="color:#87cefa;">publish</span>(channel<span style="color:#98fb98;">: string</span>, j <span style="color:#98fb98;">: RPC.Json.json</span>) =  
    <span style="color:#87cefa;">publish_client</span>(channel, <span style="color:#87cefa;">Json.to_string</span>(j))  
<span style="color:#00ffff;">}}</span>  
  
<span style="color:#00ffff;">do</span> <span style="color:#87cefa;">Resource.register_external_js</span>(<span style="color:#ffa07a;">"http://cdn.pubnub.com/pubnub-3.1.min.js"</span>)</pre>  
The first thing to notice is setting the _pubnub_publish_ variable. The _%%_ syntax is specifying that _pubnub.publish_ is the key of an external primitive. _pubnub_ is the name of the plugin (pubnub.opp) and _publish_ is the function we registered in the plugin.  
  
After this we create another variable _publish_client_ that specifies that that this function (variables can contain functions) is to be run on the client side. Lastly, we define a module _PubNub_ and a _publish_ function inside the module to be used by our Opa programs.  
  
At the end we make sure that the needed PubNub Javascript file is includes in all resources by registering it and giving the URL to the PubNub CDN.  
  
Now we can compile it all together with the _opa-plugin-builder_ command from above and use the plugin.  
<pre style="color:#00ff00;background-color:#000000;"><span style="color:#00ffff;">import</span> pubnub  
  
<span style="color:#87cefa;">broadcast</span>(author, msg) =  
   <span style="color:#b0c4de;">username </span><span style="color:#98fb98;">: RPC.Json.json</span><span style="color:#b0c4de;"> =</span> <span style="color:#00ffff;">{</span><span style="color:#b0c4de;">String =</span> author<span style="color:#00ffff;">}</span>  
   <span style="color:#b0c4de;">message </span><span style="color:#98fb98;">: RPC.Json.json</span><span style="color:#b0c4de;"> =</span> <span style="color:#00ffff;">{</span><span style="color:#b0c4de;">String =</span> msg<span style="color:#00ffff;">}</span>  
   <span style="color:#b0c4de;">record_json </span><span style="color:#98fb98;">: RPC.Json.json</span><span style="color:#b0c4de;"> =</span> <span style="color:#00ffff;">{</span><span style="color:#b0c4de;">Record =</span> [(<span style="color:#ffa07a;">"username"</span>, username), (<span style="color:#ffa07a;">"message"</span>, message)]<span style="color:#00ffff;">}</span>  
   <span style="color:#00ffff;">do</span> <span style="color:#87cefa;">PubNub.publish</span>(<span style="color:#ffa07a;">"chat"</span>, record_json)  
   <span style="color:#87cefa;">Dom.clear_value</span>(#entry)</pre>  
The _broadcast_ function is part of pubnub_chat.opa. _broadcast_ takes 2 arguments, the username of the user sending the message and the message itself. We then construct three variables of Opa's json type, _username_, _message_ and _record_json_. _username_ and _message_ are simply Json strings and they are then combined into a Json Record type that is a list of string/Json pairs. Since our _PubNub.publish_ function converts the Opa Json to a string before sending to the Javascript binding we can simply pass the Opa Json variable _record_json_.  
  
Lastly, lets take a quick look at the _subscribe_ function. Defining it in _pubnub.js_ look like:  
<pre style="color:#00ff00;background-color:#000000;"><span style="color:#ff0000;">##</span>register <span style="color:#eedd82;">subscribe :</span> string, (string -&gt; <span style="color:#7fffd4;">void</span>) -&gt; <span style="color:#7fffd4;">void</span></pre>  
Since it takes a callback function you'll notice that the second argument's type looks different from in _publish_. _(string -&gt; void)_ is defining that the function takes a variable that is a function which takes a single argument of type _string_ and has a _void_ return type.  
  
Now in _pubnub.opa_ we have a _subscribe_ function that is hardly different from the _publish_ function we described above.  
<pre style="color:#00ff00;background-color:#000000;"><span style="color:#87cefa;">subscribe</span>(channel<span style="color:#98fb98;">: string</span>, callback) =  
  <span style="color:#87cefa;">subscribe_client</span>(channel, callback)</pre>  
But how do we define a function we can pass to _subscribe_? That is the great thing, it can be any Opa function!  
<pre style="color:#00ff00;background-color:#000000;"><span style="color:#87cefa;">PubNub.subscribe</span>(<span style="color:#ffa07a;">"chat"</span>, (x <span style="color:#00ffff;">-&gt;</span> <span style="color:#87cefa;">user_update</span>(x)))</pre>  
Above we are subscribing to the channel _chat_ and passing an anonymous function that takes a single argument and sends it to the function _user_update_.  
<pre style="color:#00ff00;background-color:#000000;"><span style="color:#87cefa;">user_update</span>(x) =  
  <span style="color:#00ffff;">match</span> <span style="color:#87cefa;">Json.deserialize</span>(x) <span style="color:#00ffff;">with</span>  
         | <span style="color:#00ffff;">{</span><span style="color:#b0c4de;">some=</span><span style="color:#00ffff;">{</span><span style="color:#b0c4de;">Record =</span> [(<span style="color:#ffa07a;">"username"</span>, username), (<span style="color:#ffa07a;">"message"</span>, message)]<span style="color:#00ffff;">}}</span> <span style="color:#00ffff;">-&gt;</span>  
                    <span style="color:#b0c4de;">line =</span> <span style="color:#ffa07a;">&lt;</span><span style="color:#00ffff;">div </span><span style="color:#ffa07a;">class="line"&gt; &lt;</span><span style="color:#00ffff;">div </span><span style="color:#ffa07a;">class="user"&gt;{</span><span style="color:#87cefa;">Json.to_string</span>(username)<span style="color:#ffa07a;">}:&lt;</span><span style="color:#00ffff;">/</span><span style="color:#ffa07a;">&gt;</span>  
                             <span style="color:#ffa07a;">&lt;</span><span style="color:#00ffff;">div </span><span style="color:#ffa07a;">class="message"&gt;{</span><span style="color:#87cefa;">Json.to_string</span>(message)<span style="color:#ffa07a;">}&lt;</span><span style="color:#00ffff;">/</span><span style="color:#ffa07a;">&gt;</span>  
                            &lt;/&gt;  
                    <span style="color:#00ffff;">do</span> <span style="color:#87cefa;">Dom.transform</span>([#conversation <span style="color:#00ffff;">+&lt;-</span> line ])  
                    <span style="color:#87cefa;">Dom.scroll_to_bottom</span>(#conversation)  
         | _ <span style="color:#00ffff;">-&gt;</span> void</pre>  
Now every time a message is posted to the _chat_ channel, our callback, the anonymous function, will be run and call the function _user_update_ which we see deserializes the Json string to get an Opa Json Record. The rest is basic Opa DOM manipulation to add the new message to the messages in the conversation element.  
  
In my next post on Opa and PubNub, I'll be describing an Opa server side API for PubNub.

