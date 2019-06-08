+++
author = "Tristan Sloughter"
type="post"
categories = ["Javascript", "Opa", "Web"]
date = 2011-10-05T04:19:27Z
description = ""
draft = false
slug = "todomvc-in-opa"
tags = ["Javascript", "Opa", "Web"]
title = "TodoMVC in Opa"

+++

Edit: I just learned that [dotcloud](http://www.dotcloud.com) supports Opa! So I've pushed OpaDo and you can see a demo here [http://opado-tristan.sloughter.dotcloud.com/](http://opado-tristan.sloughter.dotcloud.com/)  
  
I wanted something quick and simple to do in [Opa](http://www.opalang.org) to give it a try so I decided to implement the TodoMVC example that has been redone in almost all Javascript frameworks, [https://github.com/addyosmani/todomvc](https://github.com/addyosmani/todomvc).  
  
The code can be found on GitHub here: [https://github.com/tsloughter/OpaDo](https://github.com/tsloughter/OpaDo)  
  
Opa is unique in that it is not only a new language but also a new web server and database. While Opa's page pushes the idea that its for the cloud and its easy distribution, I found the nicest part being the static typing and no need for Javascript.  
  
The functions below handle interactions with the Todo items. It somewhat reminds me of [Lift](http://liftweb.net/) but taken even farther.  
<pre style="color:#bebebe;background-color:#262626;"><span style="color:#ffa07a;">/** * {1</span><span style="color:#ffa07a;font-weight:bold;"> User interface</span><span style="color:#ffa07a;">} */</span>  
<span style="color:#87cefa;">update_counts</span>() =  
  <span style="color:#b0c4de;">num_done =</span> <span style="color:#87cefa;">Dom.length</span>(<span style="color:#87cefa;">Dom.select_class</span>(<span style="color:#ffa07a;">"done"</span>))  
  <span style="color:#b0c4de;">total =</span> <span style="color:#87cefa;">Dom.length</span>(<span style="color:#87cefa;">Dom.select_class</span>(<span style="color:#ffa07a;">"todo"</span>))  
  <span style="color:#00ffff;">do</span> <span style="color:#87cefa;">Dom.set_text</span>(#number_done, <span style="color:#87cefa;">Int.to_string</span>(num_done))  
  <span style="color:#87cefa;">Dom.set_text</span>(#number_left, <span style="color:#87cefa;">Int.to_string</span>(total <span style="color:#00ffff;">-</span> num_done))  
  
<span style="color:#87cefa;">make_done</span>(id<span style="color:#98fb98;">: string</span>) =  
  <span style="color:#00ffff;">do</span> <span style="color:#00ffff;">if</span> <span style="color:#87cefa;">Dom.is_checked</span>(<span style="color:#87cefa;">Dom.select_inside</span>(#<span style="color:#00ffff;">{</span>id<span style="color:#00ffff;">}</span>, <span style="color:#87cefa;">Dom.select_raw</span>(<span style="color:#ffa07a;">"input"</span>))) <span style="color:#00ffff;">then</span> <span style="color:#87cefa;">Dom.add_class</span>(#<span style="color:#00ffff;">{</span>id<span style="color:#00ffff;">}</span>, <span style="color:#ffa07a;">"done"</span>)  
  <span style="color:#00ffff;">else</span>  
    <span style="color:#87cefa;">Dom.remove_class</span>(#<span style="color:#00ffff;">{</span>id<span style="color:#00ffff;">}</span>, <span style="color:#ffa07a;">"done"</span>)  
  
  <span style="color:#87cefa;">update_counts</span>()  
  
<span style="color:#87cefa;">remove_item</span>(id<span style="color:#98fb98;">: string</span>) =  
  <span style="color:#00ffff;">do</span> <span style="color:#87cefa;">Dom.remove</span>(#<span style="color:#00ffff;">{</span>id<span style="color:#00ffff;">}</span>)  
  <span style="color:#87cefa;">update_counts</span>()  
  
<span style="color:#87cefa;">remove_all_done</span>() =  
  <span style="color:#00ffff;">do</span> <span style="color:#87cefa;">Dom.remove</span>(<span style="color:#87cefa;">Dom.select_parent_one</span>(<span style="color:#87cefa;">Dom.select_class</span>(<span style="color:#ffa07a;">"done"</span>)))  
  <span style="color:#87cefa;">update_counts</span>()  
  
<span style="color:#87cefa;">add_todo</span>(x<span style="color:#98fb98;">: string</span>) =  
  <span style="color:#b0c4de;">id =</span> <span style="color:#87cefa;">Random.string</span>(8)  
  <span style="color:#b0c4de;">li_id =</span> <span style="color:#87cefa;">Random.string</span>(8)  
  <span style="color:#b0c4de;">line =</span> <span style="color:#ffa07a;">&lt;</span><span style="color:#00ffff;">li </span><span style="color:#ffa07a;">id={</span> li_id <span style="color:#ffa07a;">}&gt;&lt;</span><span style="color:#00ffff;">div </span><span style="color:#ffa07a;">class="todo" id={</span> id <span style="color:#ffa07a;">}&gt; &lt;</span><span style="color:#00ffff;">div </span><span style="color:#ffa07a;">class="display"&gt; &lt;</span><span style="color:#00ffff;">input </span><span style="color:#ffa07a;">class="check" type="checkbox" onclick={</span>_ <span style="color:#00ffff;">-&gt;</span> <span style="color:#87cefa;">make_done</span>(id) <span style="color:#ffa07a;">} </span><span style="color:#00ffff;">/</span><span style="color:#ffa07a;">&gt; &lt;</span><span style="color:#00ffff;">div </span><span style="color:#ffa07a;">class="todo_content"&gt;{</span> x <span style="color:#ffa07a;">}&lt;</span><span style="color:#00ffff;">/div</span><span style="color:#ffa07a;">&gt; &lt;</span><span style="color:#00ffff;">span </span><span style="color:#ffa07a;">class="todo_destroy" onclick={</span>_ <span style="color:#00ffff;">-&gt;</span> <span style="color:#87cefa;">remove_item</span>(li_id) <span style="color:#ffa07a;">}&gt;&lt;</span><span style="color:#00ffff;">/span</span><span style="color:#ffa07a;">&gt; &lt;</span><span style="color:#00ffff;">/div</span><span style="color:#ffa07a;">&gt; &lt;</span><span style="color:#00ffff;">div </span><span style="color:#ffa07a;">class="edit"&gt; &lt;</span><span style="color:#00ffff;">input </span><span style="color:#ffa07a;">class="todo-input" type="text" value="" </span><span style="color:#00ffff;">/</span><span style="color:#ffa07a;">&gt; &lt;</span><span style="color:#00ffff;">/div</span><span style="color:#ffa07a;">&gt; &lt;</span><span style="color:#00ffff;">/div</span><span style="color:#ffa07a;">&gt;&lt;</span><span style="color:#00ffff;">/li</span><span style="color:#ffa07a;">&gt;</span>  
  <span style="color:#00ffff;">do</span> <span style="color:#87cefa;">Dom.transform</span>([#todo_list <span style="color:#00ffff;">+&lt;-</span> line ])  
  <span style="color:#00ffff;">do</span> <span style="color:#87cefa;">Dom.scroll_to_bottom</span>(#todo_list)  
  <span style="color:#00ffff;">do</span> <span style="color:#87cefa;">Dom.set_value</span>(#new_todo, <span style="color:#ffa07a;">""</span>)  
  <span style="color:#87cefa;">update_counts</span>()</pre>  
It is unique in combining the HTML into the language itself. Some have argued against this but when it works well it makes perfect sense. I don't want to have to convert a designers HTML into some other representation! And being able to have type checked dynamic functionality within the HTML is a boon. Even with just this simple program, I found the usefulness of the type checker outstanding.  
  
Next we have the main outline of the page and the entry part for the program.  
<pre style="color:#bebebe;background-color:#262626;"><span style="color:#87cefa;">start</span>() =  
  <span style="color:#ffa07a;">&lt;</span><span style="color:#00ffff;">div </span><span style="color:#ffa07a;">id="todoapp"&gt; &lt;</span><span style="color:#00ffff;">div </span><span style="color:#ffa07a;">class="title"&gt; &lt;</span><span style="color:#00ffff;">h1</span><span style="color:#ffa07a;">&gt;Todos&lt;</span><span style="color:#00ffff;">/h1</span><span style="color:#ffa07a;">&gt; &lt;</span><span style="color:#00ffff;">/div</span><span style="color:#ffa07a;">&gt;</span>  
    <span style="color:#ffa07a;">&lt;</span><span style="color:#00ffff;">div </span><span style="color:#ffa07a;">class="content"&gt; &lt;</span><span style="color:#00ffff;">div </span><span style="color:#ffa07a;">id=#create_todo&gt; &lt;</span><span style="color:#00ffff;">input </span><span style="color:#ffa07a;">id=#new_todo placeholder="What needs to be done?" type="text" onnewline={</span>_ <span style="color:#00ffff;">-&gt;</span> <span style="color:#87cefa;">add_todo</span>(<span style="color:#87cefa;">Dom.get_value</span>(#new_todo)) <span style="color:#ffa07a;">} </span><span style="color:#00ffff;">/</span><span style="color:#ffa07a;">&gt; &lt;</span><span style="color:#00ffff;">/div</span><span style="color:#ffa07a;">&gt;</span>  
      <span style="color:#ffa07a;">&lt;</span><span style="color:#00ffff;">div </span><span style="color:#ffa07a;">id=#todos&gt; &lt;</span><span style="color:#00ffff;">ul </span><span style="color:#ffa07a;">id=#todo_list&gt;&lt;</span><span style="color:#00ffff;">/ul</span><span style="color:#ffa07a;">&gt; &lt;</span><span style="color:#00ffff;">/div</span><span style="color:#ffa07a;">&gt;</span>  
  
      <span style="color:#ffa07a;">&lt;</span><span style="color:#00ffff;">div </span><span style="color:#ffa07a;">id="todo_stats"&gt; &lt;</span><span style="color:#00ffff;">span </span><span style="color:#ffa07a;">class="todo_count"&gt; &lt;</span><span style="color:#00ffff;">span </span><span style="color:#ffa07a;">id=#number_left class="number"&gt;0&lt;</span><span style="color:#00ffff;">/span</span><span style="color:#ffa07a;">&gt; &lt;</span><span style="color:#00ffff;">span </span><span style="color:#ffa07a;">class="word"&gt;items&lt;</span><span style="color:#00ffff;">/span</span><span style="color:#ffa07a;">&gt; left. &lt;</span><span style="color:#00ffff;">/span</span><span style="color:#ffa07a;">&gt; &lt;</span><span style="color:#00ffff;">span </span><span style="color:#ffa07a;">class="todo_clear"&gt; &lt;</span><span style="color:#00ffff;">a </span><span style="color:#ffa07a;">href="#" onclick={</span>_ <span style="color:#00ffff;">-&gt;</span> <span style="color:#87cefa;">remove_all_done</span>() <span style="color:#ffa07a;">}&gt; Clear &lt;</span><span style="color:#00ffff;">span </span><span style="color:#ffa07a;">id=#number_done class="number-done"&gt;0&lt;</span><span style="color:#00ffff;">/span</span><span style="color:#ffa07a;">&gt; completed &lt;</span><span style="color:#00ffff;">span </span><span style="color:#ffa07a;">class="word-done"&gt;items&lt;</span><span style="color:#00ffff;">/span</span><span style="color:#ffa07a;">&gt; &lt;</span><span style="color:#00ffff;">/a</span><span style="color:#ffa07a;">&gt; &lt;</span><span style="color:#00ffff;">/span</span><span style="color:#ffa07a;">&gt; &lt;</span><span style="color:#00ffff;">/div</span><span style="color:#ffa07a;">&gt;</span>  
    &lt;/div&gt;  
  &lt;/div&gt;  
  
<span style="color:#ffa07a;">/** * {1</span><span style="color:#ffa07a;font-weight:bold;"> Application</span><span style="color:#ffa07a;">} */</span>  
  
<span style="color:#ffa07a;">/** * Main entry point. */</span>  
<span style="color:#00ffff;">server</span><span style="color:#b0c4de;"> =</span> <span style="color:#87cefa;">Server.one_page_bundle</span>(<span style="color:#ffa07a;">"Todo"</span>,  
       [<span style="color:#cd3700;">@static_resource_directory</span>(<span style="color:#ffa07a;">"resources"</span>)],  
       [<span style="color:#ffa07a;">"resources/todos.css"</span>], start)</pre>  
It won't be able to replace my use of Erlang for the backend and Coffeescript for the frontend, but it looks very promising.  
  
I'll be extending this example to include persistence, sessions and users and will add posts as I complete those.

