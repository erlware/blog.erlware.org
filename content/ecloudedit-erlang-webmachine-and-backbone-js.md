+++
author = "Tristan Sloughter"
type="post"
categories = ["Backbone.js", "CouchDB", "Erlang", "Javascript", "MVC", "Web", "Webmachine"]
date = 2011-02-09T09:10:43Z
description = ""
draft = false
slug = "ecloudedit-erlang-webmachine-and-backbone-js"
tags = ["Backbone.js", "CouchDB", "Erlang", "Javascript", "MVC", "Web", "Webmachine"]
title = "eCloudEdit: Erlang, Webmachine and Backbone.js"

+++

To experiment with using a pure client-side rendering talking to an [Erlang](http://www.erlang.org "Erlang") backend I've taken [James Yu's CloudEdit tutorial](http://www.jamesyu.org/2011/01/27/cloudedit-a-backbone-js-tutorial-by-example/) an app written with [Backbone.js](http://documentcloud.github.com/backbone/) and [Rails](http://rubyonrails.org/) and converted the backend to use [Webmachine](http://webmachine.basho.com/) and [CouchDB](http://couchdb.apache.org/). You can see eCloudEdit in action [here](http://erlware.org:8080). The Backbone.js code is the same so to understand that please see James' post, here I'll describe the Erlang backend.  
  
To begin with we setup two applications, one for handling the web interaction and a second for handling the database interaction. We end up with this directory layout:  
<pre style="color:#bebebe;background-color:#262626;overflow:auto;padding:5px;">  |-eCloudEdit  
   |-bin  
   |-config  
   |-doc  
   |-lib  
   |---ece_db  
   |-----doc  
   |-----ebin  
   |-----include  
   |-----src  
   |---ece_web  
   |-----doc  
   |-----ebin  
   |-----include  
   |-----priv  
   |-------images  
   |-------javascripts  
   |---------controllers  
   |---------models  
   |---------views  
   |-------stylesheets  
   |-----src</pre>  
Under ece_web/priv is where all the html and Javascript from CloudEdit is placed, with nothing changed. To serve up the static content through Webmachine we use this [resource](https://github.com/tsloughter/eCloudEdit/blob/master/lib/ece_web/src/ece_resource_static.erl) which we won't worry about in this article. Lets look at how webmachine deciced where to route requests. This is handled by a set of dispatch rules that are passed into Webmachine on startup. We can see how this is done by looking at ece_web_sup and how the supervisor loads Webmachine:  
  
[](https://github.com/tsloughter/eCloudEdit/blob/master/lib/ece_web/src/ece_web_sup.erl)  
  
[](https://github.com/tsloughter/eCloudEdit/blob/master/lib/ece_web/src/ece_web_sup.erl)  
  
[  
<pre style="color:#bebebe;background-color:#262626;overflow:auto;padding:5px;"><span style="color:#66d9ef;">-spec</span> <span style="color:#66d9ef;">init</span>(<span style="color:#b0c4de;">list</span>()) -&gt; {<span style="color:#7fffd4;">ok</span>, {<span style="color:# fd971f;">SupFlags</span>::<span style="color:#66d9ef;">any</span>(), [<span style="color:# fd971f;">ChildSpec</span>::<span style="color:#66d9ef;">any</span>()]}} |  
                          <span style="color:#7fffd4;">ignore</span> | {<span style="color:#7fffd4;">error</span>, <span style="color:# fd971f;">Reason</span>::<span style="color:#66d9ef;">any</span>()}.  
<span style="color:# a6e22e;">init</span>([]) -&gt;  
    <span style="color:# fd971f;">WebChild</span> = {<span style="color:#7fffd4;">webmachine_mochiweb</span>,  
                {<span style="color:#7fffd4;">webmachine_mochiweb</span>, <span style="color:#7fffd4;">start</span>, [<span style="color:#66d9ef;">config</span>()]},  
                <span style="color:#7fffd4;">permanent</span>, 5000, <span style="color:#7fffd4;">worker</span>, <span style="color:#7fffd4;">dynamic</span>},  
  
    <span style="color:# fd971f;">RestartStrategy</span> = <span style="color:#7fffd4;">one_for_one</span>,  
    <span style="color:# fd971f;">MaxRestarts</span> = 3,  
    <span style="color:# fd971f;">MaxSecondsBetweenRestarts</span> = 10,  
    <span style="color:# fd971f;">SupFlags</span> = {<span style="color:# fd971f;">RestartStrategy</span>, <span style="color:# fd971f;">MaxRestarts</span>, <span style="color:# fd971f;">MaxSecondsBetweenRestarts</span>},  
  
    {<span style="color:#7fffd4;">ok</span>, {<span style="color:# fd971f;">SupFlags</span> , [<span style="color:# fd971f;">WebChild</span>]}}.  
  
<span style="color:# a6e22e;">config</span>() -&gt;  
    {<span style="color:#7fffd4;">ok</span>, <span style="color:# fd971f;">IP</span>} = <span style="color:#66d9ef;">application</span>:<span style="color:#66d9ef;">get_env</span>(<span style="color:#7fffd4;">webmachine_ip</span>),  
    {<span style="color:#7fffd4;">ok</span>, <span style="color:# fd971f;">Port</span>} = <span style="color:#66d9ef;">application</span>:<span style="color:#66d9ef;">get_env</span>(<span style="color:#7fffd4;">webmachine_port</span>),  
    {<span style="color:#7fffd4;">ok</span>, <span style="color:# fd971f;">App</span>}= <span style="color:#66d9ef;">application</span>:<span style="color:#66d9ef;">get_application</span>(),  
    <span style="color:# fd971f;">LogDir</span> = <span style="color:#66d9ef;">code</span>:<span style="color:#66d9ef;">priv_dir</span>(<span style="color:# fd971f;">App</span>) <span style="color:#ff0000;">++</span> <span style="color:#e6db74;">"/logs"</span>,  
    {<span style="color:#7fffd4;">ok</span>, <span style="color:# fd971f;">Dispatch</span>} = <span style="color:#66d9ef;">file</span>:<span style="color:#66d9ef;">consult</span>(<span style="color:#66d9ef;">filename</span>:<span style="color:#66d9ef;">join</span>([<span style="color:#66d9ef;">code</span>:<span style="color:#66d9ef;">priv_dir</span>(<span style="color:# fd971f;">App</span>), <span style="color:#e6db74;">"dispatch"</span>])),  
  
    [{<span style="color:#7fffd4;">ip</span>, <span style="color:# fd971f;">IP</span>},  
     {<span style="color:#7fffd4;">port</span>, <span style="color:# fd971f;">Port</span>},  
     {<span style="color:#7fffd4;">log_dir</span>, <span style="color:# fd971f;">LogDir</span>},  
     {<span style="color:#7fffd4;">backlog</span>, 128},  
     {<span style="color:#7fffd4;">dispatch</span>, <span style="color:# fd971f;">Dispatch</span>}].</pre>  
](https://github.com/tsloughter/eCloudEdit/blob/master/lib/ece_web/src/ece_web_sup.erl)  
  
[](https://github.com/tsloughter/eCloudEdit/blob/master/lib/ece_web/src/ece_web_sup.erl)  
  
[](https://github.com/tsloughter/eCloudEdit/blob/master/lib/ece_web/src/ece_web_sup.erl)  
  
The dispatch terms are loaded from a file _dispatch_ in the application's priv directory. For this app the dispatch file contains:  
  
[](https://github.com/tsloughter/eCloudEdit/blob/master/lib/ece_web/priv/dispatch)  
  
[](https://github.com/tsloughter/eCloudEdit/blob/master/lib/ece_web/priv/dispatch)  
  
[  
<pre style="color:#bebebe;background-color:#262626;overflow:auto;padding:5px;"><span style="background-color:#3e3d32;">{</span>[<span style="color:#e6db74;">"documents"</span>, <span style="color:#7fffd4;">id</span>], <span style="color:#7fffd4;">ece_resource_documents</span>, []<span style="background-color:#3e3d32;">}</span>.  
{[<span style="color:#e6db74;">"documents"</span>], <span style="color:#7fffd4;">ece_resource_documents</span>, []}.  
{[<span style="color:#7fffd4;">'*'</span>], <span style="color:#7fffd4;">ece_resource_static</span>, []}.</pre>  
](https://github.com/tsloughter/eCloudEdit/blob/master/lib/ece_web/priv/dispatch)  
  
[](https://github.com/tsloughter/eCloudEdit/blob/master/lib/ece_web/priv/dispatch)  
  
[](https://github.com/tsloughter/eCloudEdit/blob/master/lib/ece_web/priv/dispatch)  
  
There are two resources, one for handling the requests for creating, updating and viewing documents and one for serving up all other requests (static html, js and css files). The first rule matches paths like, _/documents/foo_, _id_ is set to _foo_ and the request is sent to the documents resource. If there is nothing after _/documents_ it is still sent to the documents resource but there is no _id_.  
  
Webmachine is essentially a REST toolkit. You build resources by defining functions that handle the different possible HTTP requests. For this webapp we'll only be dealing with GET, POST and PUT. GET is used if we'd like to retrieve information on documents, POST is for creating a new document and PUT is for updating a document.  
  
[](https://github.com/tsloughter/eCloudEdit/blob/master/lib/ece_web/src/ece_resource_documents.erl)  
  
[](https://github.com/tsloughter/eCloudEdit/blob/master/lib/ece_web/src/ece_resource_documents.erl)  
  
[  
<pre style="color:#bebebe;background-color:#262626;overflow:auto;padding:5px;"><span style="color:#66d9ef;">-record</span>(<span style="color:#66d9ef;">ctx</span>, {<span style="color:#7fffd4;">db</span>}).  
  
<span style="color:# a6e22e;">init</span>([]) -&gt;  
    {<span style="color:#7fffd4;">ok</span>, <span style="color:# fd971f;">PID</span>} = <span style="color:#66d9ef;">ece_db_sup</span>:<span style="color:#66d9ef;">start_child</span>(),  
    {<span style="color:#7fffd4;">ok</span>, #<span style="color:#66d9ef;">ctx</span>{<span style="color:#7fffd4;">db</span>=<span style="color:# fd971f;">PID</span>}}.  
  
<span style="color:# a6e22e;">allowed_methods</span>(<span style="color:# fd971f;">ReqData</span>, <span style="color:# fd971f;">Ctx</span>) -&gt;  
    {[<span style="color:#7fffd4;">'HEAD'</span>, <span style="color:#7fffd4;">'GET'</span>, <span style="color:#7fffd4;">'POST'</span>, <span style="color:#7fffd4;">'PUT'</span>], <span style="color:# fd971f;">ReqData</span>, <span style="color:# fd971f;">Ctx</span>}.  
  
<span style="color:# a6e22e;">content_types_accepted</span>(<span style="color:# fd971f;">ReqData</span>, <span style="color:# fd971f;">Ctx</span>) -&gt;  
    {[{<span style="color:#e6db74;">"application/json"</span>, <span style="color:#7fffd4;">from_json</span>}], <span style="color:# fd971f;">ReqData</span>, <span style="color:# fd971f;">Ctx</span>}.  
  
<span style="color:# a6e22e;">content_types_provided</span>(<span style="color:# fd971f;">ReqData</span>, <span style="color:# fd971f;">Ctx</span>) -&gt;  
    {[{<span style="color:#e6db74;">"application/json"</span>, <span style="color:#7fffd4;">to_json</span>}], <span style="color:# fd971f;">ReqData</span>, <span style="color:# fd971f;">Ctx</span>}.  
  
<span style="color:# a6e22e;">finish_request</span>(<span style="color:# fd971f;">ReqData</span>, <span style="color:# fd971f;">Ctx</span>) -&gt;  
    <span style="color:#66d9ef;">ece_db_sup</span>:<span style="color:#66d9ef;">terminate_child</span>(<span style="color:# fd971f;">Ctx</span>#<span style="color:#66d9ef;">ctx</span>.<span style="color:#7fffd4;">db</span>),  
    {<span style="color:#7fffd4;">true</span>, <span style="color:# fd971f;">ReqData</span>, <span style="color:# fd971f;">Ctx</span>}.</pre>  
](https://github.com/tsloughter/eCloudEdit/blob/master/lib/ece_web/src/ece_resource_documents.erl)  
  
[](https://github.com/tsloughter/eCloudEdit/blob/master/lib/ece_web/src/ece_resource_documents.erl)  
  
[](https://github.com/tsloughter/eCloudEdit/blob/master/lib/ece_web/src/ece_resource_documents.erl)  
  
To handle GET requests we implemented the _to_json_ function we specified in _content_types_provided_ to handle GET requests for json data.  
  
[](https://github.com/tsloughter/eCloudEdit/blob/master/lib/ece_web/src/ece_resource_documents.erl)  
  
[](https://github.com/tsloughter/eCloudEdit/blob/master/lib/ece_web/src/ece_resource_documents.erl)  
  
[  
<pre style="color:#bebebe;background-color:#262626;overflow:auto;padding:5px;"><span style="color:# a6e22e;">to_json</span>(<span style="color:# fd971f;">ReqData</span>, <span style="color:# fd971f;">Ctx</span>) -&gt;  
    <span style="color:#f92672;">case</span> <span style="color:#66d9ef;">wrq</span>:<span style="color:#66d9ef;">path_info</span>(<span style="color:#7fffd4;">id</span>, <span style="color:# fd971f;">ReqData</span>) <span style="color:#f92672;">of</span>  
        <span style="color:#7fffd4;">undefined</span> -&gt;  
            <span style="color:# fd971f;">All</span> = <span style="color:#66d9ef;">ece_db</span>:<span style="color:#66d9ef;">all</span>(<span style="color:# fd971f;">Ctx</span>#<span style="color:#66d9ef;">ctx</span>.<span style="color:#7fffd4;">db</span>),  
            {<span style="color:# fd971f;">All</span>, <span style="color:# fd971f;">ReqData</span>, <span style="color:# fd971f;">Ctx</span>};  
        <span style="color:# fd971f;">ID</span> -&gt;  
            <span style="color:# fd971f;">JsonDoc</span> = <span style="color:#66d9ef;">ece_db</span>:<span style="color:#66d9ef;">find</span>(<span style="color:# fd971f;">Ctx</span>#<span style="color:#66d9ef;">ctx</span>.<span style="color:#7fffd4;">db</span>, <span style="color:# fd971f;">ID</span>),  
            {<span style="color:# fd971f;">JsonDoc</span>, <span style="color:# fd971f;">ReqData</span>, <span style="color:# fd971f;">Ctx</span>}  
    <span style="color:#f92672;">end</span>.</pre>  
](https://github.com/tsloughter/eCloudEdit/blob/master/lib/ece_web/src/ece_resource_documents.erl)  
  
[](https://github.com/tsloughter/eCloudEdit/blob/master/lib/ece_web/src/ece_resource_documents.erl)  
  
[](https://github.com/tsloughter/eCloudEdit/blob/master/lib/ece_web/src/ece_resource_documents.erl)  
  
_wrq:path_info_ is used to find the value of _id_, which we set in the dispatch file, if it is _undefined_ we know the request is for all documents, while if it has a value we know to find the contents of the document with that id and return its contents. We'll see the content of _ece_db:all/1_ and _ece_db:find/2_ in the next article. Just know they both return JSON data structures.  
  
Now we must support POST for creating documents or we have nothing to return to a GET request.  
  
[](https://github.com/tsloughter/eCloudEdit/blob/master/lib/ece_web/src/ece_resource_documents.erl)  
  
[](https://github.com/tsloughter/eCloudEdit/blob/master/lib/ece_web/src/ece_resource_documents.erl)  
  
[  
<pre style="color:#bebebe;background-color:#262626;overflow:auto;padding:5px;"><span style="color:# a6e22e;">process_post</span>(<span style="color:# fd971f;">ReqData</span>, <span style="color:# fd971f;">Ctx</span>) -&gt;  
    [{<span style="color:# fd971f;">JsonDoc</span>, <span style="color:# fd971f;">_</span>}] = <span style="color:#66d9ef;">mochiweb_util</span>:<span style="color:#66d9ef;">parse_qs</span>(<span style="color:#66d9ef;">wrq</span>:<span style="color:#66d9ef;">req_body</span>(<span style="color:# fd971f;">ReqData</span>)),  
    {<span style="color:#7fffd4;">struct</span>, <span style="color:# fd971f;">Doc</span>} = <span style="color:#66d9ef;">mochijson2</span>:<span style="color:#66d9ef;">decode</span>(<span style="color:# fd971f;">JsonDoc</span>),  
    <span style="color:# fd971f;">NewDoc</span> = <span style="color:#66d9ef;">ece_db</span>:<span style="color:#66d9ef;">create</span>(<span style="color:# fd971f;">Ctx</span>#<span style="color:#66d9ef;">ctx</span>.<span style="color:#7fffd4;">db</span>, {<span style="color:# fd971f;">Doc</span>}),  
    <span style="color:# fd971f;">ReqData2</span> = <span style="color:#66d9ef;">wrq</span>:<span style="color:#66d9ef;">set_resp_body</span>(<span style="color:# fd971f;">NewDoc</span>, <span style="color:# fd971f;">ReqData</span>),  
    {<span style="color:#7fffd4;">true</span>, <span style="color:# fd971f;">ReqData2</span>, <span style="color:# fd971f;">Ctx</span>}.</pre>  
](https://github.com/tsloughter/eCloudEdit/blob/master/lib/ece_web/src/ece_resource_documents.erl)  
  
[](https://github.com/tsloughter/eCloudEdit/blob/master/lib/ece_web/src/ece_resource_documents.erl)  
  
[](https://github.com/tsloughter/eCloudEdit/blob/master/lib/ece_web/src/ece_resource_documents.erl)  
  
_wrq:req_body/1_ returns the contents of the body sent in the HTTP request. Here it is the conents of the document to store. We decode it to an Erlang data structure and pass it to the _ece_db_ app for inserting into the database. After inserting to the database the _create/2_ function returns the new document with a new element id (in this case generated by CouchDB). This is required so we know the document's id which is used by the Backbone.js frontend. In order to return it from the POST request we must set response body to the contents of the document with _wrq:set_resp_body/2_  
  
Lastly, updating documents requires support for PUT. In _contents_type_accepted/2_ we've specified that PUT requests with JSON content is sent to the function _from_json/2_:  
  
[](https://github.com/tsloughter/eCloudEdit/blob/master/lib/ece_web/src/ece_resource_documents.erl)  
  
[](https://github.com/tsloughter/eCloudEdit/blob/master/lib/ece_web/src/ece_resource_documents.erl)  
  
[  
<pre style="color:#bebebe;background-color:#262626;overflow:auto;padding:5px;"><span style="color:# a6e22e;">from_json</span>(<span style="color:# fd971f;">ReqData</span>, <span style="color:# fd971f;">Ctx</span>) -&gt;  
    <span style="color:#f92672;">case</span> <span style="color:#66d9ef;">wrq</span>:<span style="color:#66d9ef;">path_info</span>(<span style="color:#7fffd4;">id</span>, <span style="color:# fd971f;">ReqData</span>) <span style="color:#f92672;">of</span>  
        <span style="color:#7fffd4;">undefined</span> -&gt;  
            {<span style="color:#7fffd4;">false</span>, <span style="color:# fd971f;">ReqData</span>, <span style="color:# fd971f;">Ctx</span>};  
        <span style="color:# fd971f;">ID</span> -&gt;  
            <span style="color:# fd971f;">JsonDoc</span> = <span style="color:#66d9ef;">wrq</span>:<span style="color:#66d9ef;">req_body</span>(<span style="color:# fd971f;">ReqData</span>),  
            {<span style="color:#7fffd4;">struct</span>, <span style="color:# fd971f;">Doc</span>} = <span style="color:#66d9ef;">mochijson2</span>:<span style="color:#66d9ef;">decode</span>(<span style="color:# fd971f;">JsonDoc</span>),  
            <span style="color:# fd971f;">NewDoc</span> = <span style="color:#66d9ef;">ece_db</span>:<span style="color:#66d9ef;">update</span>(<span style="color:# fd971f;">Ctx</span>#<span style="color:#66d9ef;">ctx</span>.<span style="color:#7fffd4;">db</span>, <span style="color:# fd971f;">ID</span>, <span style="color:# fd971f;">Doc</span>),  
            <span style="color:# fd971f;">ReqData2</span> = <span style="color:#66d9ef;">wrq</span>:<span style="color:#66d9ef;">set_resp_body</span>(<span style="color:# fd971f;">NewDoc</span>, <span style="color:# fd971f;">ReqData</span>),  
            {<span style="color:#7fffd4;">true</span>, <span style="color:# fd971f;">ReqData2</span>, <span style="color:# fd971f;">Ctx</span>}  
    <span style="color:#f92672;">end</span>.</pre>  
](https://github.com/tsloughter/eCloudEdit/blob/master/lib/ece_web/src/ece_resource_documents.erl)  
  
[](https://github.com/tsloughter/eCloudEdit/blob/master/lib/ece_web/src/ece_resource_documents.erl)  
  
[](https://github.com/tsloughter/eCloudEdit/blob/master/lib/ece_web/src/ece_resource_documents.erl)  
  
If this request was not routed through the first rule in our dispatch file it does not have an _id_ and thus can not be an update. When this happens we return false so the frontend is aware something has gone wrong. For requests containing an _id_ we pass the contents of the requests body to _ece_db_'s _update/2_ function.  
  
In the next post I'll show how _ece_db_ is implemented with [Couchbeam](http://benoitc.github.com/couchbeam/) for reading and writing the documents to CouchDB on [Cloudant](http://www.cloudant.com).

