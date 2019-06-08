+++
author = "Tristan Sloughter"
categories = ["Javascript", "Opa", "Web"]
date = 2011-11-06T23:19:38Z
description = ""
draft = false
slug = "adding-js-to-all-opa-resources-use-case-google-analytics"
tags = ["Javascript", "Opa", "Web"]
title = "Adding JS To all Opa Resources: Use Case Google Analytics"

+++

I decided I wanted to add [Google Analytics](http://www.google.com/analytics/ "Google Analytics") to [OpaDo](http://opado.org/ "OpaDo") but had no idea how to easily tell each page to include the necessary Javascript. I asked on the Opa mailing list and got a quick and simple response. Frederic Ye pointed me to [Resource.register_external_js](http://doc.opalang.org/new_doc/#stdlib.core.web.resource.resource.opa.html/!/value_stdlib.core.web.resource.Resource.register_external_js "Opa Resource.register_external_js")  
  
It couldn't have been any easier. You simply place your google_analytics.js file in your project and use the Resource.register_external_js function to modify the default customization of all Resources. See code below or on the [github repo](https://github.com/tsloughter/opado "OpaDo Github").  
<pre style="color:#00ff00;background-color:#000000;"><span style="color:#00ffff;">package</span> opado<span style="color:#7fffd4;">.main</span>  
  
<span style="color:#00ffff;">import</span> opado<span style="color:#7fffd4;">.user</span>  
<span style="color:#00ffff;">import</span> opado<span style="color:#7fffd4;">.admin</span>  
<span style="color:#00ffff;">import</span> opado<span style="color:#7fffd4;">.todo</span>  
  
<span style="color:#b0c4de;">urls </span><span style="color:#98fb98;">: Parser.general_parser(http_request -&gt; resource)</span><span style="color:#b0c4de;"> =</span>  
  <span style="color:#00ffff;">parser</span>  
  | <span style="color:#00ffff;">{</span><span style="color:#87cefa;">Rule.debug_parse_string</span>(s <span style="color:#00ffff;">-&gt;</span> <span style="color:#87cefa;">Log.notice</span>(<span style="color:#ffa07a;">"URL"</span>,s))<span style="color:#00ffff;">}</span> Rule<span style="color:#7fffd4;">.fail</span> <span style="color:#00ffff;">-&gt;</span> <span style="color:#87cefa;">error</span>(<span style="color:#ffa07a;">""</span>)  
  | <span style="color:#ffa07a;">"/todos"</span> <span style="color:#b0c4de;">result=</span><span style="color:#00ffff;">{</span>Todo<span style="color:#7fffd4;">.resource</span><span style="color:#00ffff;">}</span> <span style="color:#00ffff;">-&gt;</span> result  
  | <span style="color:#ffa07a;">"/user"</span> <span style="color:#b0c4de;">result=</span><span style="color:#00ffff;">{</span>User<span style="color:#7fffd4;">.resource</span><span style="color:#00ffff;">}</span> <span style="color:#00ffff;">-&gt;</span> result  
  | <span style="color:#ffa07a;">"/login"</span> <span style="color:#b0c4de;">result=</span><span style="color:#00ffff;">{</span>User<span style="color:#7fffd4;">.resource</span><span style="color:#00ffff;">}</span> <span style="color:#00ffff;">-&gt;</span> result  
  | <span style="color:#ffa07a;">"/admin"</span> <span style="color:#b0c4de;">result=</span><span style="color:#00ffff;">{</span>Admin<span style="color:#7fffd4;">.resource</span><span style="color:#00ffff;">}</span> <span style="color:#00ffff;">-&gt;</span> result  
  | (.<span style="color:#00ffff;">*</span>) <span style="color:#b0c4de;">result=</span><span style="color:#00ffff;">{</span>Todo<span style="color:#7fffd4;">.resource</span><span style="color:#00ffff;">}</span> <span style="color:#00ffff;">-&gt;</span> result  
  
<span style="color:#00ffff;">do</span> <span style="color:#87cefa;">Resource.register_external_js</span>(<span style="color:#ffa07a;">"/resources/js/google_analytics.js"</span>)  
<span style="color:#00ffff;">server</span><span style="color:#b0c4de;"> =</span> <span style="color:#87cefa;">Server.of_bundle</span>([<span style="color:#cd3700;">@static_resource_directory</span>(<span style="color:#ffa07a;">"resources"</span>)])  
<span style="color:#00ffff;">server</span><span style="color:#b0c4de;"> =</span> <span style="color:#87cefa;">Server.make</span>(urls)</pre>  
For a longer article/tutorial on dealing with external resources check out this blog post from the Opa team [Dealing with External Resources](http://blog.opalang.org/2011/07/image-part-1-dealing-with-external.html "Opa: Dealing with External Resources").

