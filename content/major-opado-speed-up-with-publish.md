+++
author = "Tristan Sloughter"
type="post"
categories = ["dotcloud", "Functional Programming", "Opa", "Web"]
date = 2011-11-07T07:19:48Z
description = ""
draft = false
slug = "major-opado-speed-up-with-publish"
tags = ["dotcloud", "Functional Programming", "Opa", "Web"]
title = "Major OpaDo Speed-Up with @publish"

+++

I received a patch for [OpaDo](http://opado.org) from [Francois Regis Sinot](https://github.com/frsinot) that has made the hosted OpaDo [http://opado.org](http://opado.org) on [Dotcloud](http://dotcloud.com) MUCH faster, adding and removing item is now instantaneous. He placed @publish around explicitly server side functions that caused adding todo items to go from 21 to 1 server calls. I thought this simple but maybe not obvious for new Opa developers (it certainly wasn't to me!) needed a blog post so that beginners like my self may find it and be able to start using the @publish directive. Check out the [Opa manual](http://doc.opalang.org) for some more information on @publish, specifically I'd look at [Section 4.7.4](http://doc.opalang.org/index.html#_questions_2) and [10.2](http://doc.opalang.org/index.html#_client_server_distribution)  
  
Below are the changes he made. Which you can also of see in the [commit diff](https://github.com/tsloughter/OpaDo/commit/d57604109a10b322303e7e2e8512f518304d581c) on Github and check out the [entire source](https://github.com/tsloughter/OpaDo).  
  
  <pre style="color:#00FF00;background-color:#000000;">      
<span style="color:#00ffff;">-</span>   <span style="color:#cd3700;">@async</span>  
<span style="color:#00ffff;">-</span>   <span style="color:#87cefa;">db_make_done</span>(username<span style="color:#98fb98;">: string</span>, id<span style="color:#98fb98;">: string</span>) =  
<span style="color:#00ffff;">+</span>   <span style="color:#cd3700;">@publish</span> <span style="color:#cd3700;">@async</span>  
<span style="color:#00ffff;">+</span>   <span style="color:#87cefa;">db_make_done</span>(id<span style="color:#98fb98;">: string</span>) =  
<span style="color:#00ffff;">+</span>       <span style="color:#b0c4de;">username =</span> <span style="color:#87cefa;">User.get_username</span>()  
        <span style="color:#b0c4de;">items =</span> /todo_items[username]  
        <span style="color:#b0c4de;">item =</span> <span style="color:#87cefa;">Option.get</span>(<span style="color:#87cefa;">StringMap.get</span>(id, items))  
        /todo_items[username] <span style="color:#00ffff;">&lt;-</span> <span style="color:#87cefa;">StringMap.add</span>(id, <span style="color:#00ffff;">{</span>item <span style="color:#00ffff;">with</span> <span style="color:#b0c4de;">done=</span><span style="color:#00ffff;">true}</span>, items)  
  
<span style="color:#00ffff;">-</span>   <span style="color:#cd3700;">@async</span>  
<span style="color:#00ffff;">+</span>   <span style="color:#cd3700;">@publish</span> <span style="color:#cd3700;">@async</span>  
    <span style="color:#87cefa;">db_remove_item</span>(id<span style="color:#98fb98;">: string</span>) =  
      <span style="color:#b0c4de;">username =</span> <span style="color:#87cefa;">User.get_username</span>()  
      <span style="color:#b0c4de;">items =</span> /todo_items[username]  
  
<span style="color:#00ffff;">-</span>   <span style="color:#cd3700;">@async</span>  
<span style="color:#00ffff;">+</span>   <span style="color:#cd3700;">@publish</span> <span style="color:#cd3700;">@async</span>  
    <span style="color:#87cefa;">db_add_todo</span>(id<span style="color:#98fb98;">: string</span>, x<span style="color:#98fb98;">: string</span>) =  
      <span style="color:#b0c4de;">username =</span> <span style="color:#87cefa;">User.get_username</span>()  
      <span style="color:#b0c4de;">items =</span> /todo_items[username]  
      /todo_items[username] <span style="color:#00ffff;">&lt;-</span> <span style="color:#87cefa;">StringMap.add</span>(id, <span style="color:#00ffff;">{</span> <span style="color:#b0c4de;">value=</span>x <span style="color:#b0c4de;">done=</span><span style="color:#00ffff;">false</span> <span style="color:#b0c4de;">created_at=</span><span style="color:#ffa07a;">""</span> <span style="color:#00ffff;">}</span>, items)  
  
<span style="color:#00ffff;">+</span>   <span style="color:#cd3700;">@publish</span>  
    <span style="color:#87cefa;">add_todos</span>() =  
      <span style="color:#b0c4de;">username =</span> <span style="color:#87cefa;">User.get_username</span>()  
      <span style="color:#b0c4de;">items =</span> /todo_items[username]  
</pre>

