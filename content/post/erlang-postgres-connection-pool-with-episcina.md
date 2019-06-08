+++
author = "Tristan Sloughter"
categories = ["Databases", "Erlang", "Postgres"]
date = 2014-05-05T02:41:52Z
description = ""
draft = false
slug = "erlang-postgres-connection-pool-with-episcina"
tags = ["Databases", "Erlang", "Postgres"]
title = "Erlang Postgres Connection Pool with Episcina"

+++

Almost exactly a year ago I was looking to merge the many forks of [Will Glozer's Postgres client](https://github.com/wg/epgsql) for use in a project at Heroku. Instead [Semiocast](https://github.com/semiocast/) released their [client](https://github.com/semiocast/pgsql), I gave it a try and never looked back. (But note that David Welton, a braver person than me, is working on merging the forks of [epgsql](https://github.com/davidw/epgsql) at this time). I found Semiocast's client to be clean, stable and I liked the interface better.  
  
At the same time I was in the need of a connection pooler. Many have relied on [poolboy](https://github.com/devinus/poolboy) or [pooler](https://github.com/seth/pooler) for this purpose but neither actually fits the use case of connection pooling that well. Luckily Eric and Jordan were in the need at the same time and created the Erlware project [episcina](https://github.com/erlware/episcina), which they based off of Joseph Wecker's fork of Will Glozer's [epgsql pool](https://github.com/josephwecker/epgsql_pool). Episcina differs in that it is purely for connection pooling, it is not for pooling workers and it is not for pooling generic processes.  
  
Here I'll show how I combined the two in a [simple example](https://github.com/tsloughter/postgres_pool_example).  
  
To start we have a `sys.config` file to configure episcina:  
<pre style="line-height:1.3em;font-family:monospace;color:#f8f8f2;background-color:#272822;">{<span style="color:#f8f8f2;background-color:#272822;">episcina</span>, [{<span style="color:#f8f8f2;background-color:#272822;">pools</span>, [{<span style="color:#f8f8f2;background-color:#272822;">primary</span>,  
                        [{<span style="color:#f8f8f2;background-color:#272822;">size</span>, 10},                          
                         {<span style="color:#f8f8f2;background-color:#272822;">timeout</span>, 10000},  
                         {<span style="color:#f8f8f2;background-color:#272822;">connect_provider</span>, {<span style="color:#f8f8f2;background-color:#272822;">pp_db</span>, <span style="color:#f8f8f2;background-color:#272822;">open</span>,  
                                             [[{<span style="color:#f8f8f2;background-color:#272822;">host</span>, <span style="color:#e6db74;">"localhost"</span>}  
                                              ,{<span style="color:#f8f8f2;background-color:#272822;">database</span>, <span style="color:#e6db74;">"postgres_pool"</span>}  
                                              ,{<span style="color:#f8f8f2;background-color:#272822;">port</span>, 5432}  
                                              ,{<span style="color:#f8f8f2;background-color:#272822;">user</span>, <span style="color:#e6db74;">"postgres"</span>}  
                                              ,{<span style="color:#f8f8f2;background-color:#272822;">password</span>, <span style="color:#e6db74;">"password"</span>}]]}},  
                         {<span style="color:#f8f8f2;background-color:#272822;">close_provider</span>, {<span style="color:#f8f8f2;background-color:#272822;">pp_db</span>, <span style="color:#f8f8f2;background-color:#272822;">close</span>, []}}]}]  
              }  
             ]  
}  
  
</pre>  
A key thing to note here is the connect and close providers are function calls to modules within the project and not the Postgres client. Episcina requires a return value of `{ok, pid()}` and the Semiocast client returns `{pgsql_connection, pid()}`, so we wrap the connection calls to get around that:  
<pre style="line-height:1.3em;font-family:monospace;color:#f8f8f2;background-color:#272822;"><span style="color:#f92672;">-spec</span> <span style="color:#66d9ef;">get_connection</span>(<span style="color:#66d9ef;">atom</span>()) <span style="color:#a6e22e;">-&gt;</span> {<span style="color:#f8f8f2;background-color:#272822;">pgsql_connection</span>, <span style="color:#66d9ef;">pid</span>()} | {<span style="color:#f8f8f2;background-color:#272822;">error</span>, <span style="color:#f8f8f2;background-color:#272822;">timeout</span>}.  
<span style="color:#a6e22e;">get_connection</span>(<span style="color:#fd971f;">Pool</span>) <span style="color:#a6e22e;">-&gt;</span>  
    <span style="color:#f92672;">case</span> <span style="color:#66d9ef;">episcina</span>:<span style="color:#66d9ef;">get_connection</span>(<span style="color:#fd971f;">Pool</span>) <span style="color:#f92672;">of</span>  
        {<span style="color:#f8f8f2;background-color:#272822;">ok</span>, <span style="color:#fd971f;">Pid</span>} <span style="color:#a6e22e;">-&gt;</span>  
            {<span style="color:#f8f8f2;background-color:#272822;">pgsql_connection</span>, <span style="color:#fd971f;">Pid</span>};  
        {<span style="color:#f8f8f2;background-color:#272822;">error</span>, <span style="color:#f8f8f2;background-color:#272822;">timeout</span>} <span style="color:#a6e22e;">-&gt;</span>  
            {<span style="color:#f8f8f2;background-color:#272822;">error</span>, <span style="color:#f8f8f2;background-color:#272822;">timeout</span>}  
    <span style="color:#f92672;">end</span>.  
  
<span style="color:#f92672;">-spec</span> <span style="color:#66d9ef;">return_connection</span>(<span style="color:#66d9ef;">atom</span>(), {<span style="color:#f8f8f2;background-color:#272822;">pgsql_connection</span>, <span style="color:#66d9ef;">pid</span>()}) <span style="color:#a6e22e;">-&gt;</span> <span style="color:#f8f8f2;background-color:#272822;">ok</span>.  
<span style="color:#a6e22e;">return_connection</span>(<span style="color:#fd971f;">Pool</span>, {<span style="color:#f8f8f2;background-color:#272822;">pgsql_connection</span>, <span style="color:#fd971f;">Pid</span>}) <span style="color:#a6e22e;">-&gt;</span>  
    <span style="color:#66d9ef;">episcina</span>:<span style="color:#66d9ef;">return_connection</span>(<span style="color:#fd971f;">Pool</span>, <span style="color:#fd971f;">Pid</span>).  
  
<span style="color:#f92672;">-spec</span> <span style="color:#66d9ef;">open</span>(<span style="color:#66d9ef;">list</span>()) <span style="color:#a6e22e;">-&gt;</span> {<span style="color:#f8f8f2;background-color:#272822;">ok</span>, <span style="color:#66d9ef;">pid</span>()}.  
<span style="color:#a6e22e;">open</span>(<span style="color:#fd971f;">DBArgs</span>) <span style="color:#a6e22e;">-&gt;</span>  
    {<span style="color:#f8f8f2;background-color:#272822;">pgsql_connection</span>, <span style="color:#fd971f;">Pid</span>} = <span style="color:#66d9ef;">pgsql_connection</span>:<span style="color:#66d9ef;">open</span>(<span style="color:#fd971f;">DBArgs</span>),  
    {<span style="color:#f8f8f2;background-color:#272822;">ok</span>, <span style="color:#fd971f;">Pid</span>}.  
  
<span style="color:#f92672;">-spec</span> <span style="color:#66d9ef;">close</span>(<span style="color:#66d9ef;">pid</span>()) <span style="color:#a6e22e;">-&gt;</span> <span style="color:#f8f8f2;background-color:#272822;">ok</span>.  
<span style="color:#a6e22e;">close</span>(<span style="color:#fd971f;">Pid</span>) <span style="color:#a6e22e;">-&gt;</span>  
    <span style="color:#66d9ef;">pgsql_connection</span>:<span style="color:#66d9ef;">close</span>({<span style="color:#f8f8f2;background-color:#272822;">pgsql_connection</span>, <span style="color:#fd971f;">Pid</span>}).  
</pre>  
And here is the query function to get a connection and return it to the pool after completion:  
<pre style="line-height:1.3em;font-family:monospace;color:#f8f8f2;background-color:#272822;"><span style="color:#f92672;">  
-spec</span> <span style="color:#66d9ef;">query</span>(<span style="color:#66d9ef;">string</span>()) <span style="color:#a6e22e;">-&gt;</span> <span style="color:#66d9ef;">tuple</span>().  
<span style="color:#a6e22e;">query</span>(<span style="color:#fd971f;">Query</span>) <span style="color:#a6e22e;">-&gt;</span>  
    <span style="color:#fd971f;">C</span> = <span style="color:#66d9ef;">get_connection</span>(<span style="color:#f8f8f2;background-color:#272822;">primary</span>),  
    <span style="color:#f92672;">try</span>  
        <span style="color:#66d9ef;">pgsql_connection</span>:<span style="color:#66d9ef;">simple_query</span>(<span style="color:#fd971f;">Query</span>, [], <span style="color:#f8f8f2;background-color:#272822;">infinity</span>, <span style="color:#fd971f;">C</span>)  
    <span style="color:#f92672;">after</span>  
        <span style="color:#66d9ef;">return_connection</span>(<span style="color:#f8f8f2;background-color:#272822;">primary</span>, <span style="color:#fd971f;">C</span>)  
    <span style="color:#f92672;">end</span>.  
</pre>  
This example project uses [relx](http://relx.org) to build a release which will start episcina on boot:  
<pre style="line-height:1.3em;font-family:monospace;color:#f8f8f2;background-color:#272822;">{release, {postgres_pool, "0.0.1"},  
  [postgres_pool]}.  
  
{sys_config, "./config/sys.config"}.  
{dev_mode, true}.  
  
{include_erts, true}.  
{extended_start_script, true}.  
</pre>  
Boot the release to an interactive shell and play around:  
  
<pre style="line-height:1.3em;font-family:monospace;color:#f8f8f2;background-color:#272822;">  
λ _rel/bin/postgres_pool console  
(postgres_pool@127.0.0.1)1&gt; pp_db:query("SELECT 1").  
{{select,1},[{1}]}  
</pre>

