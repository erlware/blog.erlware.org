+++
author = "Tristan Sloughter"
type="post"
categories = ["CouchDB", "Erlang", "simple_one_for_one", "Web", "Webmachine"]
date = 2011-02-12T23:37:34Z
description = ""
draft = false
slug = "ecloudedit-part-2-couchdb"
tags = ["CouchDB", "Erlang", "simple_one_for_one", "Web", "Webmachine"]
title = "eCloudEdit Part 2: CouchDB"

+++

In my [last post](http://blog.erlware.org/2011/02/08/ecloudedit-erlang-webmachine-and-backbone-js/) I showed the Webmachine backend to James Yu's CloudEdit app in Backbone.js. What was left out was, where are the documents stored? Here I'll show how this is done with CouchDB. And you can give the app a try at [http://erlware.org:8080](http://erlware.org:8080)  
  
First, a new Erlang app is needed that we'll call ece_db.  
  
<pre style="color:#bebebe;background-color:#262626;overflow:auto;padding:5px;">  
ece_db/  
&#9500;&#9472;&#9472; doc  
&#9500;&#9472;&#9472; ebin  
&#9474;&#160;&#160; &#9500;&#9472;&#9472; ece_db.app  
&#9474;&#160;&#160; &#9492;&#9472;&#9472; overview.edoc  
&#9500;&#9472;&#9472; include  
&#9492;&#9472;&#9472; src  
    &#9500;&#9472;&#9472; ece_db.erl  
    &#9500;&#9472;&#9472; ece_db_app.erl  
    &#9492;&#9472;&#9472; ece_db_sup.erl</pre>  
  
Three modules are implemented, one that starts the app by calling the supervisor's start function, the supervisor itself that sets up a _simple_one_for_one_ and the _gen_server_ that handles the frontends requests for creating and modifying documents in CouchDB.  
  
We'll ignore the _ece_db_app.erl_ module for this post and start with _ece_db_sup.erl_. On requests from the Webmachine resource module we don't want one to wait on the other, requests should be handled in parrallel. One option is to not create a process for the database backend and instead have all of the functions in the database interface run in the process of the Webmachine resource. However, two reasons to not do this are that there are multiple pieces of data that must be read out of a configuration file and used to setup what is needed to talk to CouchDB. We do not want to be doing this for every request! Furthermore a supervised _gen_server_ allows us the possibility of retrying requests with no extra code and crashing without retrying but not taking down the resource's process that is handling the user's HTTP request. It is a lot nicer and easier to just let things fail when we can!  
  
<pre style="color:#bebebe;background-color:#262626;overflow:auto;padding:5px;">  
  <span style="background-color:#3E3D32;">{</span><span style="color:#7fffd4;">ece_db</span>, [{<span style="color:#7fffd4;">server</span>, <span style="color:#E6DB74;">"HOSTNAME"</span>},  
            {<span style="color:#7fffd4;">port</span>, 80},  
            {<span style="color:#7fffd4;">database</span>, <span style="color:#E6DB74;">"ecloudedit"</span>}]<span style="background-color:#3E3D32;">}</span></pre>  
  
<pre style="color:#bebebe;background-color:#262626;overflow:auto;padding:5px;">  
<span style="color:#66d9ef;">-spec</span> <span style="color:#66d9ef;">init</span>(<span style="color:#b0c4de;">list</span>()) -&gt; {<span style="color:#7fffd4;">ok</span>, {<span style="color:# FD971F;">SupFlags</span>::<span style="color:#66d9ef;">any</span>(), [<span style="color:# FD971F;">ChildSpec</span>::<span style="color:#66d9ef;">any</span>()]}} |  
                       <span style="color:#7fffd4;">ignore</span> | {<span style="color:#7fffd4;">error</span>, <span style="color:# FD971F;">Reason</span>::<span style="color:#66d9ef;">any</span>()}.  
<span style="color:# A6E22E;">init</span>([]) -&gt;  
    <span style="color:# FD971F;">RestartStrategy</span> = <span style="color:#7fffd4;">simple_one_for_one</span>,  
    <span style="color:# FD971F;">MaxRestarts</span> = 0,  
    <span style="color:# FD971F;">MaxSecondsBetweenRestarts</span> = 1,  
  
    <span style="color:# FD971F;">SupFlags</span> = {<span style="color:# FD971F;">RestartStrategy</span>, <span style="color:# FD971F;">MaxRestarts</span>, <span style="color:# FD971F;">MaxSecondsBetweenRestarts</span>},  
  
    <span style="color:# FD971F;">Restart</span> = <span style="color:#7fffd4;">temporary</span>,  
    <span style="color:# FD971F;">Shutdown</span> = 2000,  
    <span style="color:# FD971F;">Type</span> = <span style="color:#7fffd4;">worker</span>,  
  
    {<span style="color:#7fffd4;">ok</span>, <span style="color:# FD971F;">Server</span>} = <span style="color:#66d9ef;">application</span>:<span style="color:#66d9ef;">get_env</span>(<span style="color:#7fffd4;">server</span>),  
    {<span style="color:#7fffd4;">ok</span>, <span style="color:# FD971F;">Port</span>} = <span style="color:#66d9ef;">application</span>:<span style="color:#66d9ef;">get_env</span>(<span style="color:#7fffd4;">port</span>),  
    {<span style="color:#7fffd4;">ok</span>, <span style="color:# FD971F;">DB</span>} = <span style="color:#66d9ef;">application</span>:<span style="color:#66d9ef;">get_env</span>(<span style="color:#7fffd4;">database</span>),  
  
    <span style="color:# FD971F;">AChild</span> = {<span style="color:#7fffd4;">ece_db</span>, {<span style="color:#7fffd4;">ece_db</span>, <span style="color:#7fffd4;">start_link</span>, [<span style="color:# FD971F;">Server</span>, <span style="color:# FD971F;">Port</span>, <span style="color:# FD971F;">DB</span>]},  
              <span style="color:# FD971F;">Restart</span>, <span style="color:# FD971F;">Shutdown</span>, <span style="color:# FD971F;">Type</span>, [<span style="color:#7fffd4;">ece_db</span>]},  
  
    {<span style="color:#7fffd4;">ok</span>, {<span style="color:# FD971F;">SupFlags</span>, [<span style="color:# FD971F;">AChild</span>]}}.</pre>  
  
First, we have the config entries in config/sys.config that will be read in by _application:gen_env_. Here we set the server URL, port number and name of the database to use. CouchDB can easily be run locally but for my running copy I use a database hosted by the great service [Cloudant](http://www.cloudant.com). Next is the _init_ function for _ece_db_sup.erl_. A key thing to note for a _simple_one_for_one_ is that NO process is started after the supervisor's init function returns like with other types of supervisor children. Instead we must explicitly call _start_child_. This is how we are able to create a supervised _gen_server_ process for every HTTP request. Below is the code in _ece_db_sup_ for starting and stopping the process:  
  
<pre style="color:#bebebe;background-color:#262626;overflow:auto;padding:5px;">  
<span style="color:# A6E22E;">start_child</span>() -&gt;  
    <span style="color:#66d9ef;">supervisor</span>:<span style="color:#66d9ef;">start_child</span>(?<span style="color:#66d9ef;">SERVER</span>, []).  
  
<span style="color:# A6E22E;">terminate_child</span>(<span style="color:# FD971F;">PID</span>) -&gt;  
    <span style="color:#66d9ef;">ece_db</span>:<span style="color:#66d9ef;">terminate_child</span>(<span style="color:# FD971F;">PID</span>).</pre>  
  
In the last post I showed the functions that start and stop the _ece_db_ process. Here they are again:  
  
<pre style="color:#bebebe;background-color:#262626;overflow:auto;padding:5px;">  
<span style="color:# A6E22E;">init</span>([]) -&gt;  
    {<span style="color:#7fffd4;">ok</span>, <span style="color:# FD971F;">PID</span>} = <span style="color:#66d9ef;">ece_db_sup</span>:<span style="color:#66d9ef;">start_child</span>(),  
    {<span style="color:#7fffd4;">ok</span>, #<span style="color:#66d9ef;">ctx</span>{<span style="color:#7fffd4;">db</span>=<span style="color:# FD971F;">PID</span>}}.</pre>  
  
<pre style="color:#bebebe;background-color:#262626;overflow:auto;padding:5px;">  
<span style="color:# A6E22E;">finish_request</span>(<span style="color:# FD971F;">ReqData</span>, <span style="color:# FD971F;">Ctx</span>) -&gt;  
    <span style="color:#66d9ef;">ece_db_sup</span>:<span style="color:#66d9ef;">terminate_child</span>(<span style="color:# FD971F;">Ctx</span>#<span style="color:#66d9ef;">ctx</span>.<span style="color:#7fffd4;">db</span>),  
    {<span style="color:#7fffd4;">true</span>, <span style="color:# FD971F;">ReqData</span>, <span style="color:# FD971F;">Ctx</span>}.</pre>  
  
On each request Webmachine calls the _init_ function for the resource that is matched in the dispatch table. In that init function we call _start_child_ in _ece_db_sup_ which returns _{ok, PID}_. The PID is the process id we'll be sending messages. Now in _ece_db_ we implement the API functions needed to start the process and to interact with it.  
  
<pre style="color:#bebebe;background-color:#262626;overflow:auto;padding:5px;">  
<span style="color:# A6E22E;">start_link</span>(<span style="color:# FD971F;">Server</span>, <span style="color:# FD971F;">Port</span>, <span style="color:# FD971F;">DB</span>) -&gt;  
    <span style="color:#66d9ef;">gen_server</span>:<span style="color:#66d9ef;">start_link</span>(?<span style="color:#66d9ef;">MODULE</span>, [<span style="color:# FD971F;">Server</span>, <span style="color:# FD971F;">Port</span>, <span style="color:# FD971F;">DB</span>], []).  
  
<span style="color:# A6E22E;">all</span>(<span style="color:# FD971F;">PID</span>) -&gt;  
    <span style="color:#66d9ef;">gen_server</span>:<span style="color:#66d9ef;">call</span>(<span style="color:# FD971F;">PID</span>, <span style="color:#7fffd4;">all</span>).  
  
<span style="color:# A6E22E;">find</span>(<span style="color:# FD971F;">PID</span>, <span style="color:# FD971F;">ID</span>) -&gt;  
    <span style="color:#66d9ef;">gen_server</span>:<span style="color:#66d9ef;">call</span>(<span style="color:# FD971F;">PID</span>, {<span style="color:#7fffd4;">find</span>, <span style="color:# FD971F;">ID</span>}).  
  
<span style="color:# A6E22E;">create</span>(<span style="color:# FD971F;">PID</span>, <span style="color:# FD971F;">Doc</span>) -&gt;  
    <span style="color:#66d9ef;">gen_server</span>:<span style="color:#66d9ef;">call</span>(<span style="color:# FD971F;">PID</span>, {<span style="color:#7fffd4;">create</span>, <span style="color:# FD971F;">Doc</span>}).  
  
<span style="color:# A6E22E;">update</span>(<span style="color:# FD971F;">PID</span>, <span style="color:# FD971F;">ID</span>, <span style="color:# FD971F;">JsonDoc</span>) -&gt;  
    <span style="color:#66d9ef;">gen_server</span>:<span style="color:#66d9ef;">call</span>(<span style="color:# FD971F;">PID</span>, {<span style="color:#7fffd4;">update</span>, <span style="color:# FD971F;">ID</span>, <span style="color:# FD971F;">JsonDoc</span>}).  
  
<span style="color:# A6E22E;">terminate</span>(<span style="color:# FD971F;">PID</span>) -&gt;  
    <span style="color:#66d9ef;">gen_server</span>:<span style="color:#66d9ef;">call</span>(<span style="color:# FD971F;">PID</span>, <span style="color:#7fffd4;">terminate</span>).  
</pre>  
  
Each function, besides the one for starting the process, takes a PID. This PID is the process _gen_server:call_ to which it sends its message. To handle these messages we have the _gen_server_ callbacks.  
  
<pre style="color:#bebebe;background-color:#262626;overflow:auto;padding:5px;">  
<span style="color:#75715E;">%%%</span><span style="color:#75715E;">===================================================================  
</span><span style="color:#75715E;">%%% </span><span style="color:#75715E;">gen_server callbacks  
</span><span style="color:#75715E;">%%%</span><span style="color:#75715E;">===================================================================  
</span>  
<span style="color:#75715E;">%% </span><span style="color:#75715E;">@private  
</span><span style="color:# A6E22E;">init</span>([<span style="color:# FD971F;">Server</span>, <span style="color:# FD971F;">Port</span>, <span style="color:# FD971F;">DB</span>]) -&gt;  
    <span style="color:# FD971F;">CouchServer</span> = <span style="color:#66d9ef;">couchbeam</span>:<span style="color:#66d9ef;">server_connection</span>(<span style="color:# FD971F;">Server</span>, <span style="color:# FD971F;">Port</span>, <span style="color:#E6DB74;">""</span>, []),  
    {<span style="color:#7fffd4;">ok</span>, <span style="color:# FD971F;">CouchDB</span>} = <span style="color:#66d9ef;">couchbeam</span>:<span style="color:#66d9ef;">open_db</span>(<span style="color:# FD971F;">CouchServer</span>, <span style="color:# FD971F;">DB</span>),  
    {<span style="color:#7fffd4;">ok</span>, #<span style="color:#66d9ef;">state</span>{<span style="color:#7fffd4;">db</span>=<span style="color:# FD971F;">CouchDB</span>}}.  
  
<span style="color:#75715E;">%% </span><span style="color:#75715E;">@private  
</span><span style="color:# A6E22E;">handle_call</span>(<span style="color:#7fffd4;">all</span>, <span style="color:# FD971F;">_From</span>, #<span style="color:#66d9ef;">state</span>{<span style="color:#7fffd4;">db</span>=<span style="color:# FD971F;">DB</span>}=<span style="color:# FD971F;">State</span>) -&gt;  
    <span style="color:# FD971F;">Docs</span> = <span style="color:#66d9ef;">get_docs</span>(<span style="color:# FD971F;">DB</span>, [{<span style="color:#7fffd4;">descending</span>, <span style="color:#7fffd4;">true</span>}]),  
    {<span style="color:#7fffd4;">reply</span>, <span style="color:#66d9ef;">mochijson2</span>:<span style="color:#66d9ef;">encode</span>(<span style="color:# FD971F;">Docs</span>), <span style="color:# FD971F;">State</span>};  
<span style="color:# A6E22E;">handle_call</span>({<span style="color:#7fffd4;">find</span>, <span style="color:# FD971F;">ID</span>}, <span style="color:# FD971F;">_From</span>, #<span style="color:#66d9ef;">state</span>{<span style="color:#7fffd4;">db</span>=<span style="color:# FD971F;">DB</span>}=<span style="color:# FD971F;">State</span>) -&gt;  
    [<span style="color:# FD971F;">Doc</span>] = <span style="color:#66d9ef;">get_docs</span>(<span style="color:# FD971F;">DB</span>, [{<span style="color:#7fffd4;">key</span>, <span style="color:#b0c4de;">list_to_binary</span>(<span style="color:# FD971F;">ID</span>)}]),  
    {<span style="color:#7fffd4;">reply</span>, <span style="color:#66d9ef;">mochijson2</span>:<span style="color:#66d9ef;">encode</span>(<span style="color:# FD971F;">Doc</span>), <span style="color:# FD971F;">State</span>};  
<span style="color:# A6E22E;">handle_call</span>({<span style="color:#7fffd4;">create</span>, <span style="color:# FD971F;">Doc</span>}, <span style="color:# FD971F;">_From</span>, #<span style="color:#66d9ef;">state</span>{<span style="color:#7fffd4;">db</span>=<span style="color:# FD971F;">DB</span>}=<span style="color:# FD971F;">State</span>) -&gt;  
    {<span style="color:#7fffd4;">ok</span>, <span style="color:# FD971F;">Doc1</span>} = <span style="color:#66d9ef;">couchbeam</span>:<span style="color:#66d9ef;">save_doc</span>(<span style="color:# FD971F;">DB</span>, <span style="color:# FD971F;">Doc</span>),  
    {<span style="color:# FD971F;">NewDoc</span>} = <span style="color:#66d9ef;">couchbeam_doc</span>:<span style="color:#66d9ef;">set_value</span>(&lt;&lt;<span style="color:#E6DB74;">"id"</span>&gt;&gt;, <span style="color:#66d9ef;">couchbeam_doc</span>:<span style="color:#66d9ef;">get_id</span>(<span style="color:# FD971F;">Doc1</span>), <span style="color:# FD971F;">Doc1</span>),  
    {<span style="color:#7fffd4;">reply</span>, <span style="color:#66d9ef;">mochijson2</span>:<span style="color:#66d9ef;">encode</span>(<span style="color:# FD971F;">NewDoc</span>), <span style="color:# FD971F;">State</span>};  
<span style="color:# A6E22E;">handle_call</span>({<span style="color:#7fffd4;">update</span>, <span style="color:# FD971F;">ID</span>, <span style="color:# FD971F;">NewDoc</span>}, <span style="color:# FD971F;">_From</span>, #<span style="color:#66d9ef;">state</span>{<span style="color:#7fffd4;">db</span>=<span style="color:# FD971F;">DB</span>}=<span style="color:# FD971F;">State</span>) -&gt;  
    <span style="color:# FD971F;">IDBinary</span> = <span style="color:#b0c4de;">list_to_binary</span>(<span style="color:# FD971F;">ID</span>),  
    {<span style="color:#7fffd4;">ok</span>, <span style="color:# FD971F;">Doc</span>} = <span style="color:#66d9ef;">couchbeam</span>:<span style="color:#66d9ef;">open_doc</span>(<span style="color:# FD971F;">DB</span>, <span style="color:# FD971F;">IDBinary</span>),  
    <span style="color:# FD971F;">NewDoc2</span> = <span style="color:#66d9ef;">couchbeam_doc</span>:<span style="color:#66d9ef;">set_value</span>(&lt;&lt;<span style="color:#E6DB74;">"_id"</span>&gt;&gt;, <span style="color:# FD971F;">IDBinary</span>, {<span style="color:# FD971F;">NewDoc</span>}),  
    <span style="color:# FD971F;">NewDoc3</span> = <span style="color:#66d9ef;">couchbeam_doc</span>:<span style="color:#66d9ef;">set_value</span>(&lt;&lt;<span style="color:#E6DB74;">"_rev"</span>&gt;&gt;, <span style="color:#66d9ef;">couchbeam_doc</span>:<span style="color:#66d9ef;">get_rev</span>(<span style="color:# FD971F;">Doc</span>), <span style="color:# FD971F;">NewDoc2</span>),  
    {<span style="color:#7fffd4;">ok</span>, {<span style="color:# FD971F;">Doc1</span>}} = <span style="color:#66d9ef;">couchbeam</span>:<span style="color:#66d9ef;">save_doc</span>(<span style="color:# FD971F;">DB</span>, <span style="color:# FD971F;">NewDoc3</span>),  
    {<span style="color:#7fffd4;">reply</span>, <span style="color:#66d9ef;">mochijson2</span>:<span style="color:#66d9ef;">encode</span>(<span style="color:# FD971F;">Doc1</span>), <span style="color:# FD971F;">State</span>;  
<pre style="color:#bebebe;background-color:#262626;overflow:auto;padding:5px;"><span style="color:# A6E22E;">handle_call</span>(<span style="color:#7fffd4;">terminate</span>, <span style="color:# FD971F;">_From</span>, <span style="color:# FD971F;">State</span>) -&gt;  
    {<span style="color:#7fffd4;">stop</span>, <span style="color:#7fffd4;">normal</span>, <span style="color:# FD971F;">State</span>}.</pre>  
</pre>  
  
_init_ takes the server URL, the port number and the name of the database to store the documents as arguments and creates a [CouchBeam](http://benoitc.github.com/couchbeam/) database record. To handle the other messages we have the _handle_call_ function to match on the different tuples sent to _gen_server:call_ for the different function calls. _all_ uses the internal function _get_docs_ with the option to have the function in descending order (so the newest is first) and returns those after encoding to JSON. _find_ uses the same function but only wants a certain document which in the case of CouchDB is specified with a key value. _create_ first saves the new document which returns the document that is actually stored in CouchDB. CouchDB adds the __id_ and __rev_ key/value pairs it generates. Since our frontend expects the id to be the value of a key _id_ and not __id_ we use _couchbeam_doc:set_value_ to set a value of the new key _id_ with the value of the documents id retrieved from the document with _couchbeam_doc:get_id_. For _update_ we first open the document corresponding to the id passed in as an argument to get the __rev_ value. __rev_ needs to be set in the document we send to _save_doc_ so CouchDB knows this is a valid update based on the previous version of the document. Thus we set the new documents __id_ and __rev_ values and send to _save_doc_.  
  
Lastly, we have the _get_docs_ function. Since the frontend expects the key _id_ to exist and CouchDB stores the id with the key __id_ we use a simple view to return the documents with _id_ containing the value of the documents __id_. Views are defined with map/reduce functions that CouchDB runs across the databases documents and builds an index from. These indexes are then queried based on keys to find documents. Thus access is very quick, indexes are built with BTrees, and a map/reduce does not have to be run on the documents for every request. Additionally, when a new document is created the view's map/reduce functions only need to run over the new documents to update the BTree. Updates to the view's indexes are either done on each use of the view or can be manually told to run.  
  
<pre style="color:#bebebe;background-color:#262626;overflow:auto;padding:5px;">  
<span style="color:#F92672;">function</span> (<span style="color:#2e8b57;">doc</span>)  
{  
  emit(doc._id, {<span style="color:# FD971F;">id</span> : doc._id, <span style="color:# FD971F;">title</span> : doc.title, <span style="color:# FD971F;">created_at</span> : doc.created_at, <span style="color:# FD971F;">body</span> : doc.body});  
}</pre>  
  
<pre style="color:#bebebe;background-color:#262626;overflow:auto;padding:5px;">  
<span style="color:# A6E22E;">get_docs</span>(<span style="color:# FD971F;">DB</span>, <span style="color:# FD971F;">Options</span>) -&gt;  
    {<span style="color:#7fffd4;">ok</span>, <span style="color:# FD971F;">AllDocs</span>} = <span style="color:#66d9ef;">couchbeam</span>:<span style="color:#66d9ef;">view</span>(<span style="color:# FD971F;">DB</span>, {<span style="color:#E6DB74;">"all"</span>, <span style="color:#E6DB74;">"find"</span>}, <span style="color:# FD971F;">Options</span>),  
    {<span style="color:#7fffd4;">ok</span>, <span style="color:# FD971F;">Results</span>} = <span style="color:#66d9ef;">couchbeam_view</span>:<span style="color:#66d9ef;">fetch</span>(<span style="color:# FD971F;">AllDocs</span>),  
  
    {[{&lt;&lt;<span style="color:#E6DB74;">"total_rows"</span>&gt;&gt;, <span style="color:# FD971F;">_Total</span>},  
      {&lt;&lt;<span style="color:#E6DB74;">"offset"</span>&gt;&gt;, <span style="color:# FD971F;">_Offset</span>},  
      {&lt;&lt;<span style="color:#E6DB74;">"rows"</span>&gt;&gt;, <span style="color:# FD971F;">Rows</span>}]} = <span style="color:# FD971F;">Results</span>,  
  
    <span style="color:#66d9ef;">lists</span>:<span style="color:#66d9ef;">map</span>(<span style="color:#F92672;">fun</span>({<span style="color:# FD971F;">Row</span>}) -&gt;  
                      {&lt;&lt;<span style="color:#E6DB74;">"value"</span>&gt;&gt;, {<span style="color:# FD971F;">Value</span>}} = <span style="color:#66d9ef;">lists</span>:<span style="color:#66d9ef;">keyfind</span>(&lt;&lt;<span style="color:#E6DB74;">"value"</span>&gt;&gt;, 1, <span style="color:# FD971F;">Row</span>),  
                      <span style="color:# FD971F;">Value</span>  
              <span style="color:#F92672;">end</span>, <span style="color:# FD971F;">Rows</span>).  
</pre>  
  
In _get_docs_ the _couchbeam:view_ function is used to construct a view request to be sent to the CouchDB server. _couchbeam_view:fetch_ sends the view request and returns the results as Erlang terms. After this we extract the rows from the results and use _lists:map_ to extract out just the value of each row to be returned.  
  
That's it! There are places that could use improvement. One being the use of _lists:map_ over every returned row to construct documents the frontend can deal with. Additionally, the need to duplicate __id_ as _id_ for the frontends use could be removed through modifications on the frontend.  
  
In the next installment I'll be updating the code -- and maybe making some of these performance enhancements -- with James Yu's recent changes in his [Part 2](http://www.jamesyu.org/2011/02/09/backbone.js-tutorial-with-rails-part-2/).

