+++
author = "Tristan Sloughter"
type="post"
categories = ["Erlang", "pubnub", "pubsub", "simple_one_for_one", "Web"]
date = 2011-04-10T20:33:19Z
description = ""
draft = false
slug = "erlang-pubnub-client-and-chat"
tags = ["Erlang", "pubnub", "pubsub", "simple_one_for_one", "Web"]
title = "Erlang PubNub Client and Chat"

+++

I was thoroughly impressed with [PubNub](http://pubnub.com), a publish/subscribe service, when I first read their articles and played around with it some in Javascript. But obviously I need an Erlang API if I'm going to really use it! So I've created [ePubNub](https://github.com/tsloughter/epubnub).  
  
In the ePubNub README you'll find information on some basic usage of the application. You don't have to do anything more than use the _epubnub.erl_ module to publish and subscribe (by either providing a _PID_ to send messages to or a function handler to process each).  
  
Here I've built a little more complicated app/release called epubnub_chat, and the source is also on [github](https://github.com/tsloughter/epubnub_chat).  
  
The first thing we need is the epubnub app as a dependency in the _epubnub_chat.app_ file:  
<pre style="color:#bebebe;background-color:#262626;overflow:auto;padding:5px;">   {<span style="color:#7fffd4;">applications</span>, [<span style="color:#7fffd4;">kernel</span>, <span style="color:#7fffd4;">stdlib</span>, <span style="color:#7fffd4;">epubnub</span>]},</pre>  
We'll use a simple_one_for_one for supervising channel subscribed processes. In e_pubnub_chat_sup_ we have 3 API functions for the user to use (_start_link_ is run by the __app.erl_ module on startup): _connect/1_, _connect/2_, _disconnect/1_:  
<pre style="color:#bebebe;background-color:#262626;overflow:auto;padding:5px;"><span style="color:# a6e22e;">connect</span>(<span style="color:# fd971f;">Channel</span>) -&gt;  
    <span style="color:#66d9ef;">supervisor</span>:<span style="color:#66d9ef;">start_child</span>(?<span style="color:#66d9ef;">SERVER</span>, [<span style="color:# fd971f;">Channel</span>]).  
  
<span style="color:# a6e22e;">connect</span>(<span style="color:# fd971f;">EPN</span>, <span style="color:# fd971f;">Channel</span>) -&gt;  
    <span style="color:#66d9ef;">supervisor</span>:<span style="color:#66d9ef;">start_child</span>(?<span style="color:#66d9ef;">SERVER</span>, [<span style="color:# fd971f;">EPN</span>, <span style="color:# fd971f;">Channel</span>]).  
  
<span style="color:# a6e22e;">disconnect</span>(<span style="color:# fd971f;">PID</span>) -&gt;  
    <span style="color:#66d9ef;">epubnub_chat</span>:<span style="color:#66d9ef;">stop</span>(<span style="color:# fd971f;">PID</span>).</pre>  
EPN is a record containing the necessary url and keys for talking to the PubNub service and is created with the new functions in epubnub:  
<pre style="color:#bebebe;background-color:#262626;overflow:auto;padding:5px;"><span style="color:#66d9ef;">-spec</span> <span style="color:#66d9ef;">new</span>() -&gt; <span style="color:#b0c4de;">record</span>(<span style="color:#7fffd4;">epn</span>).  
<span style="color:# a6e22e;">new</span>() -&gt;  
    #<span style="color:#66d9ef;">epn</span>{}.  
  
<span style="color:#66d9ef;">-spec</span> <span style="color:#66d9ef;">new</span>(<span style="color:#66d9ef;">string</span>()) -&gt; <span style="color:#b0c4de;">record</span>(<span style="color:#7fffd4;">epn</span>).  
<span style="color:# a6e22e;">new</span>(<span style="color:# fd971f;">Origin</span>) -&gt;  
    #<span style="color:#66d9ef;">epn</span>{<span style="color:#7fffd4;">origin</span>=<span style="color:# fd971f;">Origin</span>}.  
  
<span style="color:#66d9ef;">-spec</span> <span style="color:#66d9ef;">new</span>(<span style="color:#66d9ef;">string</span>(), <span style="color:#66d9ef;">string</span>()) -&gt; <span style="color:#b0c4de;">record</span>(<span style="color:#7fffd4;">epn</span>).  
<span style="color:# a6e22e;">new</span>(<span style="color:# fd971f;">PubKey</span>, <span style="color:# fd971f;">SubKey</span>) -&gt;  
    #<span style="color:#66d9ef;">epn</span>{<span style="color:#7fffd4;">pubkey</span>=<span style="color:# fd971f;">PubKey</span>, <span style="color:#7fffd4;">subkey</span>=<span style="color:# fd971f;">SubKey</span>}.  
  
<span style="color:#66d9ef;">-spec</span> <span style="color:#66d9ef;">new</span>(<span style="color:#66d9ef;">string</span>(), <span style="color:#66d9ef;">string</span>(), <span style="color:#66d9ef;">string</span>()) -&gt; <span style="color:#b0c4de;">record</span>(<span style="color:#7fffd4;">epn</span>).  
<span style="color:# a6e22e;">new</span>(<span style="color:# fd971f;">Origin</span>, <span style="color:# fd971f;">PubKey</span>, <span style="color:# fd971f;">SubKey</span>) -&gt;  
    #<span style="color:#66d9ef;">epn</span>{<span style="color:#7fffd4;">origin</span>=<span style="color:# fd971f;">Origin</span>, <span style="color:#7fffd4;">pubkey</span>=<span style="color:# fd971f;">PubKey</span>, <span style="color:#7fffd4;">subkey</span>=<span style="color:# fd971f;">SubKey</span>}.</pre>  
You can pass none to _connect/1_ and it will send none to the chat gen_server and it will use defaults of pubsub.pubnub.com, demo and demo, for the url, publish key and subscribe key respectively.  
  
Now in the _epubnub_chat_ gen_server we need the following API functions:  
<pre style="color:#bebebe;background-color:#262626;overflow:auto;padding:5px;"><span style="color:# a6e22e;">start_link</span>(<span style="color:# fd971f;">Channel</span>) -&gt;  
    <span style="color:#66d9ef;">start_link</span>(<span style="color:#66d9ef;">epubnub</span>:<span style="color:#66d9ef;">new</span>(), <span style="color:# fd971f;">Channel</span>).  
  
<span style="color:# a6e22e;">start_link</span>(<span style="color:# fd971f;">EPN</span>, <span style="color:# fd971f;">Channel</span>) -&gt;  
    <span style="color:#66d9ef;">gen_server</span>:<span style="color:#66d9ef;">start_link</span>(?<span style="color:#66d9ef;">MODULE</span>, [<span style="color:# fd971f;">EPN</span>, <span style="color:# fd971f;">Channel</span>], []).  
  
<span style="color:# a6e22e;">message</span>(<span style="color:# fd971f;">Server</span>, <span style="color:# fd971f;">Msg</span>) -&gt;  
    <span style="color:#66d9ef;">gen_server</span>:<span style="color:#66d9ef;">cast</span>(<span style="color:# fd971f;">Server</span>, {<span style="color:#7fffd4;">message</span>, <span style="color:# fd971f;">Msg</span>}).  
  
<span style="color:# a6e22e;">stop</span>(<span style="color:# fd971f;">Server</span>) -&gt;  
    <span style="color:#66d9ef;">gen_server</span>:<span style="color:#66d9ef;">cast</span>(<span style="color:# fd971f;">Server</span>, <span style="color:#7fffd4;">stop</span>).</pre>  
The _start_link_ functions are run when the supervisor spawns a simple_one_for_one supervisor for the process. This returns _{ok, PID}_. This _PID_ must be remembered so we can talk to the process we have started thats subscribed to a specific channel. We pass this _PID_ to the _message/2_ and _stop/1_ functions, which we'll see at the and when we use the program.  
  
_start_link/1_ and _/2_ call _init/1_ with the provided arguments:  
<pre style="color:#bebebe;background-color:#262626;overflow:auto;padding:5px;"><span style="color:# a6e22e;">init</span>([<span style="color:# fd971f;">EPN</span>, <span style="color:# fd971f;">Channel</span>]) -&gt;  
    {<span style="color:#7fffd4;">ok</span>, <span style="color:# fd971f;">PID</span>} = <span style="color:#66d9ef;">epubnub_sup</span>:<span style="color:#66d9ef;">subscribe</span>(<span style="color:# fd971f;">EPN</span>, <span style="color:# fd971f;">Channel</span>, <span style="color:#b0c4de;">self</span>()),  
    {<span style="color:#7fffd4;">ok</span>, #<span style="color:#66d9ef;">state</span>{<span style="color:#7fffd4;">epn</span>=<span style="color:# fd971f;">EPN</span>, <span style="color:#7fffd4;">pid</span>=<span style="color:# fd971f;">PID</span>, <span style="color:#7fffd4;">channel</span>=<span style="color:# fd971f;">Channel</span>}}.</pre>  
Here we use the epubnub_sup subscribe/3 function and not epubnub:subscribe because we want it to be supervised. We store the PID for this process so we can terminate it later.  
  
The epubnub subscribe process was given the _PID_, returned by _self/1_, of the current process which is the gen_server process and will send messages that are published to the channel to that process. We handle these messages in _handle_info/2_:  
<pre style="color:#bebebe;background-color:#262626;overflow:auto;padding:5px;"><span style="color:# a6e22e;">handle_info</span>({<span style="color:#7fffd4;">message</span>, <span style="color:# fd971f;">Message</span>}, <span style="color:# fd971f;">State</span>) -&gt;  
    <span style="color:#66d9ef;">io</span>:<span style="color:#66d9ef;">format</span>(<span style="color:#e6db74;">"~p~n"</span>, [<span style="color:# fd971f;">Message</span>]),  
    {<span style="color:#7fffd4;">noreply</span>, <span style="color:# fd971f;">State</span>}.</pre>  
Lastly, we have to handle the messages from _message/2_ and _stop/1_:  
<pre style="color:#bebebe;background-color:#262626;overflow:auto;padding:5px;"><span style="color:# a6e22e;">handle_cast</span>({<span style="color:#7fffd4;">message</span>, <span style="color:# fd971f;">Msg</span>}, <span style="color:# fd971f;">State</span>=#<span style="color:#66d9ef;">state</span>{<span style="color:#7fffd4;">epn</span>=<span style="color:# fd971f;">EPN</span>, <span style="color:#7fffd4;">channel</span>=<span style="color:# fd971f;">Channel</span>}) -&gt;  
    <span style="color:#66d9ef;">epubnub</span>:<span style="color:#66d9ef;">publish</span>(<span style="color:# fd971f;">EPN</span>, <span style="color:# fd971f;">Channel</span>, <span style="color:# fd971f;">Msg</span>),  
    {<span style="color:#7fffd4;">noreply</span>, <span style="color:# fd971f;">State</span>}.  
<span style="color:#75715e;">  
</span><span style="color:# a6e22e;">terminate</span>(<span style="color:# fd971f;">_Reason</span>, #<span style="color:#66d9ef;">state</span>{<span style="color:#7fffd4;">pid</span>=<span style="color:# fd971f;">PID</span>}) -&gt;  
    <span style="color:#66d9ef;">epubnub</span>:<span style="color:#66d9ef;">unsubscribe</span>(<span style="color:# fd971f;">PID</span>),  
    <span style="color:#7fffd4;">ok</span>.</pre>  
The _handle_cast/2_ function published your message to the channel this process subscribed to with _epubnub:publish/3_ and terminate calls _epubnub:unsubscribe/1_ before this process ending which sends a terminate message to the subscibred process.  
  
Now lets see this program in action:  
<pre>[tristan@marx ~/Devel/epubnub_chat]  
09:55 (master)$ sinan dist  
starting: depends  
starting: build  
starting: release  
starting: dist  
[tristan@marx ~/Devel/epubnub_chat]  
09:55 (master)$ sudo faxien ir  
Password:  
Do you want to install the release: /Users/tristan/Devel/epubnub_chat/_build/development/tar/epubnub_chat-0.0.1.tar.gz  
Enter (y)es, (n)o, or yes to (a)ll? &gt; ? y  
Replacing existing executable file at: /usr/local/lib/erlang/bin/epubnub_chat  
Replacing existing executable file at: /usr/local/lib/erlang/bin/5.8.2/epubnub_chat  
Replacing existing executable file at: /usr/local/lib/erlang/bin/erlware_release_start_helper  
Replacing existing executable file at: /usr/local/lib/erlang/bin/5.8.2/erlware_release_start_helper  
ok  
[tristan@marx ~/Devel/epubnub_chat]  
09:56 (master)$ epubnub_chat  
....  
Eshell V5.8.2  (abort with ^G)  
1&gt; {ok, Server} = epubnub_chat_sup:connect("chat").  
{ok,}  
2&gt;  
=PROGRESS REPORT==== 10-Apr-2011::09:57:14 ===  
          supervisor: {local,inet_gethost_native_sup}  
             started: [{pid,},{mfa,{inet_gethost_native,init,[[]]}}]  
  
=PROGRESS REPORT==== 10-Apr-2011::09:57:14 ===  
          supervisor: {local,kernel_safe_sup}  
             started: [{pid,},  
                       {name,inet_gethost_native_sup},  
                       {mfargs,{inet_gethost_native,start_link,[]}},  
                       {restart_type,temporary},  
                       {shutdown,1000},  
                       {child_type,worker}]  
  
2&gt; epubnub_chat:message(Server, &lt;"hello there!"&gt;).  
ok  
3&gt; &lt;"hello there!"&gt;  
&lt;"I'm from the webapp!"&gt;  
  
3&gt; q().  
ok</pre>  
You can go to the [PubNub tutorial page](http://www.pubnub.com/blog/build-real-time-web-apps-easy) to chat between yourself, or get someone else to join!  
  
That's it! Simple and quick global communication that scales for you!  
  
I've really enjoyed playing with PubNub and hope I get to use it for a real project soon.

