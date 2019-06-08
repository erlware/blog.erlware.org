+++
author = "Tristan Sloughter"
categories = ["batman.js", "Cowboy", "Erlang", "Javascript", "RESTful", "web development", "Webmachine"]
date = 2012-04-30T19:59:11Z
description = ""
draft = false
slug = "cowboy-and-batman-js-for-erlang-web-development"
tags = ["batman.js", "Cowboy", "Erlang", "Javascript", "RESTful", "web development", "Webmachine"]
title = "Cowboy and Batman.js for Erlang Web Development"

+++

## Why Cowboy and Batman.js  
  
There are a lot of Erlang web frameworks out there today. Not all are modeled after the MVC model (see [Nitrogen](http://nitrogenproject.com/ "Nitrogen")), but I think all of them are addressing the problem the wrong way. I recently gave a presentation, [slides here](http://tsloughter.github.com/ceug_4_2012/ "Chicago Erlang User Group April 4th, 2012") and the [code for this example here](https://github.com/tsloughter/bcmvc/ "Batman Cowboy MVC"), describing my perferred method for using Erlang for web development and why I think it is the best way to go. In this post, I'll go into more details on how to build the Erlang backend for the TodoMVC clone I did with [Batman.js](http://batmanjs.org/ "Batman.js"). I will not spend time on Batman.js but instead only give a quick list of reasons I prefer it to other Javascript frameworks.  
  
Batman.js advantages:  
  
*   Automatic URL generation based on model  
*   HTML data-bind templates  
*   Coffeescript  
[Cowboy](https://github.com/extend/cowboy "Cowboy: Erlang web server") is a newer Erlang web server that provides a REST handler based on [Webmachine](https://github.com/basho/webmachine "Webmachine"). Both of these are perfect for developing a RESTful API, because they follow the HTTP standard exactly and when you are building an API based on HTTP, being able to properly reason about how the logic of the application maps to the protocol eases development and eases getting REST "right".  
  
## Nginx  
  
Any non-dynamic content should be served by [Nginx](http://nginx.com/ "Nginx") since there is no logic needed and it is something Nginx is great at, so why have Erlang do it? The snippet below configures Nginx to listen on port 80 and serve files from bcmvc's _priv_ directory. Each request is checked to see if it is a POST or any other method with a JSON request type. If either of those are true, the request is proxied on to a server listening on port 8080, in our case the Cowboy server.  
<pre style="color:#dcdccc;background-color:#3f3f3f;">server {  
  listen 80;  
  server_name localhost;  
  
  location / {  
    root   &lt;PATH TO CLONE&gt;/bcmvc/lib/bcmvc_web/priv/;  
  
    <span style="color:#f0dfaf;font-weight:bold;">if</span> ($request_method ~* POST) {  
      proxy_pass        http:<span style="color:#7f9f7f;">//</span><span style="color:#e0cf9f;font-weight:bold;text-decoration:underline;">localhost</span><span style="color:#7f9f7f;">:8080; </span>    }  
  
    <span style="color:#f0dfaf;font-weight:bold;">if</span> ($http_accept ~* application/json) {  
      proxy_pass        http:<span style="color:#7f9f7f;">//</span><span style="color:#e0cf9f;font-weight:bold;text-decoration:underline;">localhost</span><span style="color:#7f9f7f;">:8080; </span>    }  
  }  
}</pre>  
  
## The API  
  
Batman.js knows what endpoints to use and what data to send based on the name of the model we created and the encoded variables, [code here](https://github.com/tsloughter/bcmvc/blob/master/lib/bcmvc_web/priv/models/todo.coffee "Todo Model"). This results in the following API:  
<table border="2" rules="groups" cellspacing="0" cellpadding="6"><caption> </caption><col class="left" /> <col class="left" /> <col class="left" /> <col class="left" />  
<thead>  
<tr>  
<th class="left" scope="col">Method</th>  
<th class="left" scope="col">Endpoint</th>  
<th class="left" scope="col">Data</th>  
<th class="left" scope="col">Return</th>  
</tr>  
</thead>  
<tbody>  
<tr>  
<td class="left">POST</td>  
<td class="left">_todos_</td>  
<td class="left">{todo : {body:"bane wants to meet, not worried",isDone:false}}</td>  
<td class="left"></td>  
</tr>  
<tr>  
<td class="left">PUT</td>  
<td class="left">/todos/33e93b30-2371-4071-afc5-2d48226d5dba</td>  
<td class="left">{todo : {body:"bane wants to meet, not worried",isDone:false}}</td>  
<td class="left"></td>  
</tr>  
<tr>  
<td class="left">GET</td>  
<td class="left">_todos_</td>  
<td class="left"></td>  
<td class="left">[{todo : {id:"33e93b30-2371-4071-afc5-2d48226d5dba", body:"bane wants to meet, not worried", isDone:false}}]</td>  
</tr>  
<tr>  
<td class="left">DELETE</td>  
<td class="left">/todos/33e93b30-2371-4071-afc5-2d48226d5dba</td>  
<td class="left"></td>  
<td class="left"></td>  
</tr>  
</tbody>  
</table>  
  
## Cowboy Dispatch and Supervisor  
  
Dispatch rules are matched by Cowboy to know what handler to send the request to. Here we have two rules. One that matches just the URL _/todos_ and one that matches the URL with an additional element which will be associated with the atom _todo_. Both requests will be sent to the module _bcmvc_todo_handler_.  
<pre style="color:#dcdccc;background-color:#3f3f3f;">Dispatch = [{<span style="color:#cc9393;">'_'</span>, [{[&lt;&lt;<span style="color:#cc9393;">"todos"</span>&gt;&gt;], bcmvc_todo_handler, []},  
                   {[&lt;&lt;<span style="color:#cc9393;">"todos"</span>&gt;&gt;, todo], bcmvc_todo_handler, []}]}],</pre>  
Cowboy provides a useful function _child_spec_ for creating a child specfication to use in our supervisor. The child spec here tells Cowboy we want a TCP listener on port 8080 that handles the HTTP protocol. We additionally provide our dispatch list for it to match against and pass on requests.  
<pre style="color:#dcdccc;background-color:#3f3f3f;">ChildSpec = <span style="color:#8cd0d3;">cowboy</span>:<span style="color:#8cd0d3;">child_spec</span>(bcmvc_cowboy, 100, cowboy_tcp_transport,   
                              [{port, 8080}], cowboy_http_protocol, [{dispatch, <span style="color:#dfaf8f;">Dispatch</span>}]),</pre>  
  
## Cowboy Handler  
  
Now that we have a server on port 8080 that knows to send certain requests to our todo handler, we can build the module. The first required function to export is _init/3_. This function let's Cowboy know we have a REST protocol, this is how it knows what functions to call (some have defaults and some existing in our module) to handle the request.  
<pre style="color:#dcdccc;background-color:#3f3f3f;"><span style="color:#8cd0d3;">init</span>(<span style="color:#dfaf8f;">_Transport</span>, <span style="color:#dfaf8f;">_Req</span>, <span style="color:#dfaf8f;">_Opts</span>) -&gt;  
    {upgrade, protocol, cowboy_http_rest}.</pre>  
Knowing that this is a REST handler Cowboy will pass the request on to _allowed_methods/2_ to find out if our handler is able to handle this method. Next, the content types accepted and provided by the handler are checked against the incoming request. The expected HTTP response status codes are returned if any of these fail. 405 for _allowed_methods_, XXX for _content_types_accepted_ and XXX for _content_types_provided_.  
<pre style="color:#dcdccc;background-color:#3f3f3f;"><span style="color:#8cd0d3;">allowed_methods</span>(<span style="color:#dfaf8f;">Req</span>, <span style="color:#dfaf8f;">State</span>) -&gt;  
    {[<span style="color:#cc9393;">'HEAD'</span>, <span style="color:#cc9393;">'GET'</span>, <span style="color:#cc9393;">'PUT'</span>, <span style="color:#cc9393;">'POST'</span>, <span style="color:#cc9393;">'DELETE'</span>], <span style="color:#dfaf8f;">Req</span>, <span style="color:#dfaf8f;">State</span>}.  
  
<span style="color:#8cd0d3;">content_types_accepted</span>(<span style="color:#dfaf8f;">Req</span>, <span style="color:#dfaf8f;">State</span>) -&gt;  
    {[{{&lt;&lt;<span style="color:#cc9393;">"application"</span>&gt;&gt;, &lt;&lt;<span style="color:#cc9393;">"json"</span>&gt;&gt;, []}, put_json}], <span style="color:#dfaf8f;">Req</span>, <span style="color:#dfaf8f;">State</span>}.  
  
<span style="color:#8cd0d3;">content_types_provided</span>(<span style="color:#dfaf8f;">Req</span>, <span style="color:#dfaf8f;">State</span>) -&gt;  
    {[{{&lt;&lt;<span style="color:#cc9393;">"application"</span>&gt;&gt;, &lt;&lt;<span style="color:#cc9393;">"json"</span>&gt;&gt;, []}, get_json}], <span style="color:#dfaf8f;">Req</span>, <span style="color:#dfaf8f;">State</span>}.</pre>  
Now the request is sent to the function that handles the HTTP method type of the request.  
  
For a POST, a request to create a new todo item, the function _process_post/2_ is sent the request. Here we retrieve the body, a JSON object, from the request, convert it to a record and save the model. We'll see how this record conversion is done when we look at the model module. To inform the frontend of the id of our new resource we set the location header to be the path with the id.  
<pre style="color:#dcdccc;background-color:#3f3f3f;"><span style="color:#8cd0d3;">process_post</span>(<span style="color:#dfaf8f;">Req</span>, <span style="color:#dfaf8f;">State</span>) -&gt;  
    {ok, <span style="color:#dfaf8f;">Body</span>, <span style="color:#dfaf8f;">Req1</span>} = <span style="color:#8cd0d3;">cowboy_http_req</span>:<span style="color:#8cd0d3;">body</span>(<span style="color:#dfaf8f;">Req</span>),  
    <span style="color:#dfaf8f;">Todo</span> = <span style="color:#8cd0d3;">bcmvc_model_todo</span>:<span style="color:#8cd0d3;">to_record</span>(<span style="color:#dfaf8f;">Body</span>),  
    <span style="color:#8cd0d3;">bcmvc_model_todo</span>:<span style="color:#8cd0d3;">save</span>(<span style="color:#dfaf8f;">Todo</span>),  
  
    <span style="color:#dfaf8f;">NewId</span> = <span style="color:#8cd0d3;">bcmvc_model_todo</span>:<span style="color:#8cd0d3;">get</span>(id, <span style="color:#dfaf8f;">Todo</span>),  
    {ok, <span style="color:#dfaf8f;">Req2</span>} = <span style="color:#8cd0d3;">cowboy_http_req</span>:<span style="color:#8cd0d3;">set_resp_header</span>(  
                   &lt;&lt;<span style="color:#cc9393;">"Location"</span>&gt;&gt;, &lt;&lt;<span style="color:#cc9393;">"/todos/"</span>, <span style="color:#dfaf8f;">NewId</span>/binary&gt;&gt;, <span style="color:#dfaf8f;">Req1</span>),  
  
    {true, <span style="color:#dfaf8f;">Req2</span>, <span style="color:#dfaf8f;">State</span>}.</pre>  
For this handler we expect PUT for an update to an object, that is what Batman.js does, but a PATCH would make more sense. For a PUT the URL contains the id for the todo item to be updated. That is retrieved with the_ binding/2_ function. The todo record is created the same as in _process_post/2_ but then the this id is set for the model and the _update/1_ function is used to save it to the database.  
<pre style="color:#dcdccc;background-color:#3f3f3f;"><span style="color:#8cd0d3;">put_json</span>(<span style="color:#dfaf8f;">Req</span>, <span style="color:#dfaf8f;">State</span>) -&gt;  
    {ok, <span style="color:#dfaf8f;">Body</span>, <span style="color:#dfaf8f;">Req1</span>} = <span style="color:#8cd0d3;">cowboy_http_req</span>:<span style="color:#8cd0d3;">body</span>(<span style="color:#dfaf8f;">Req</span>),  
    {<span style="color:#dfaf8f;">TodoId</span>, <span style="color:#dfaf8f;">Req2</span>} = <span style="color:#8cd0d3;">cowboy_http_req</span>:<span style="color:#8cd0d3;">binding</span>(todo, <span style="color:#dfaf8f;">Req1</span>),  
    <span style="color:#dfaf8f;">Todo</span> = <span style="color:#8cd0d3;">bcmvc_model_todo</span>:<span style="color:#8cd0d3;">to_record</span>(<span style="color:#dfaf8f;">Body</span>),  
    <span style="color:#dfaf8f;">Todo2</span> = <span style="color:#8cd0d3;">bcmvc_model_todo</span>:<span style="color:#8cd0d3;">set</span>([{id, <span style="color:#dfaf8f;">TodoId</span>}], <span style="color:#dfaf8f;">Todo</span>),  
    <span style="color:#8cd0d3;">bcmvc_model_todo</span>:<span style="color:#8cd0d3;">update</span>(<span style="color:#dfaf8f;">Todo2</span>),      
    {true, <span style="color:#dfaf8f;">Req2</span>, <span style="color:#dfaf8f;">State</span>}.</pre>  
For a GET request, which for this application we do not deal with a request for a single todo item, all todo items are retrieved from the model module. Each of these is passed to the model's _to_json/1_ function and the result of converting each to JSON is combined into a binary string and placed between brackets so the Batman.js frontend receives a proper JSON list of JSON objects.  
<pre style="color:#dcdccc;background-color:#3f3f3f;"><span style="color:#8cd0d3;">get_json</span>(<span style="color:#dfaf8f;">Req</span>, <span style="color:#dfaf8f;">State</span>) -&gt;  
    <span style="color:#dfaf8f;">JsonModels</span> = <span style="color:#8cd0d3;">lists</span>:<span style="color:#8cd0d3;">foldr</span>(<span style="color:#f0dfaf;font-weight:bold;">fun</span>(<span style="color:#dfaf8f;">X</span>, &lt;&lt;<span style="color:#cc9393;">""</span>&gt;&gt;) -&gt;  
                                 <span style="color:#dfaf8f;">X</span>;  
                            (<span style="color:#dfaf8f;">X</span>, <span style="color:#dfaf8f;">Acc</span>) -&gt;  
                                 &lt;&lt;<span style="color:#dfaf8f;">Acc</span>/binary, <span style="color:#cc9393;">","</span>, <span style="color:#dfaf8f;">X</span>/binary&gt;&gt;  
                         <span style="color:#f0dfaf;font-weight:bold;">end</span>, &lt;&lt;<span style="color:#cc9393;">""</span>&gt;&gt;, [<span style="color:#8cd0d3;">bcmvc_model_todo</span>:<span style="color:#8cd0d3;">to_json</span>(<span style="color:#dfaf8f;">Model</span>) <span style="color:#f0dfaf;font-weight:bold;">||</span> <span style="color:#dfaf8f;">Model</span> <span style="color:#f0dfaf;font-weight:bold;">&lt;-</span> <span style="color:#8cd0d3;">bcmvc_model_todo</span>:<span style="color:#8cd0d3;">all</span>()]),  
  
    {&lt;&lt;<span style="color:#cc9393;">"["</span>, <span style="color:#dfaf8f;">JsonModels</span>/binary, <span style="color:#cc9393;">"]"</span>&gt;&gt;, <span style="color:#dfaf8f;">Req</span>, <span style="color:#dfaf8f;">State</span>}.</pre>  
And lastly, DELETE. Like in PUT the todo item's id is retrieved from the bindings created based on the dispatch rules and this is passed to the model's delete function.  
<pre style="color:#dcdccc;background-color:#3f3f3f;"><span style="color:#8cd0d3;">delete_resource</span>(<span style="color:#dfaf8f;">Req</span>, <span style="color:#dfaf8f;">State</span>) -&gt;  
    {<span style="color:#dfaf8f;">TodoId</span>, <span style="color:#dfaf8f;">Req1</span>} = <span style="color:#8cd0d3;">cowboy_http_req</span>:<span style="color:#8cd0d3;">binding</span>(todo, <span style="color:#dfaf8f;">Req</span>),  
    <span style="color:#8cd0d3;">bcmvc_model_todo</span>:<span style="color:#8cd0d3;">delete</span>(<span style="color:#dfaf8f;">TodoId</span>),  
    {true, <span style="color:#dfaf8f;">Req1</span>, <span style="color:#dfaf8f;">State</span>}.</pre>  
  
## Models  
  
Model's are repsented as records and must provide serialization functions to go between JSON and a record. Each model uses a parse transform that creates functions for creating and updating the record. The transform is a modified version of [exprecs](https://github.com/uwiger/parse_trans "Parse Transforms") from Ulf Wiger that also uses the type definitions in the record to ensure when setting a field that it is the correct type. For example in the todo model _isDone_ is a boolean, so when the model is created the boolean convert function will be matched to convert the string representation to an atom:  
<pre style="color:#dcdccc;background-color:#3f3f3f;"><span style="color:#8cd0d3;">convert</span>(boolean, &lt;&lt;<span style="color:#cc9393;">"false"</span>&gt;&gt;) -&gt;  
    false;  
<span style="color:#8cd0d3;">convert</span>(boolean, &lt;&lt;<span style="color:#cc9393;">"true"</span>&gt;&gt;) -&gt;  
    true;</pre>  
So the key pieces of the_ bcmvc_model_todo_ are:  
<pre style="color:#dcdccc;background-color:#3f3f3f;"><span style="color:#8cd0d3;">-compile</span>({parse_transform, bcmvc_model_transform}).  
  
<span style="color:#8cd0d3;">-record</span>(<span style="color:#8cd0d3;">bcmvc_model_todo</span>, {id = <span style="color:#8cd0d3;">ossp_uuid</span>:<span style="color:#8cd0d3;">make</span>(v1, text) :: <span style="color:#8cd0d3;">string</span>(),  
                           body                          :: <span style="color:#8cd0d3;">binary</span>(),  
                           isDone                        :: <span style="color:#8cd0d3;">boolean</span>()}).  
  
<span style="color:#8cd0d3;">to_json</span>(<span style="color:#dfaf8f;">Record</span>) -&gt;  
    ?<span style="color:#bfebbf;">record_to_json</span>(?<span style="color:#bfebbf;">MODULE</span>, <span style="color:#dfaf8f;">Record</span>).  
  
<span style="color:#8cd0d3;">to_record</span>(<span style="color:#dfaf8f;">JSON</span>) -&gt;  
    ?<span style="color:#bfebbf;">json_to_record</span>(?<span style="color:#bfebbf;">MODULE</span>, <span style="color:#dfaf8f;">JSON</span>).</pre>  
The_ ?record_to_json_ and _?json_to_record_ macros are defined in _jsonerl.hrl_. These marcos are generic and work for any record that is typed and uses the model transform.  
  
## Conclusion  
  
Clearly, much of what the resource handler and model do is generic and can be abstracted out so that implementing new models and resources can be even simpler. This is the goal of my project [Maru](https://github.com/tsloughter/maru "Maru"). Currently it is based on Webmachine but is now being convered to Cowboy.  
  
In the end, using Cowboy for building a RESTful interface for your application allows you to build interfaces for the frontend entirely separted from backend development, and if you want multiple interfaces (like native mobile and web), they both talk directly to the same backend. Also, from the beginning you have the option to open up your application with an API for other developers to take your application new places, and, shameless plug here, add your API to [Mashape](http://www.mashape.com "Mashape") to spread your new app!

