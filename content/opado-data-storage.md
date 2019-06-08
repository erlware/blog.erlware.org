+++
author = "Tristan Sloughter"
categories = ["cloud", "dotcloud", "Opa", "Web"]
date = 2011-10-06T18:59:55Z
description = ""
draft = false
slug = "opado-data-storage"
tags = ["cloud", "dotcloud", "Opa", "Web"]
title = "OpaDo Data Storage"

+++

[OpaDo](https://github.com/tsloughter/OpaDo) (a port of the TodoMVC app to Opa) now persists todo items to the Opa database. The new version is up on [dotcloud](http://www.dotcloud.com), [http://opado-tristan.sloughter.dotcloud.com/](http://opado-tristan.sloughter.dotcloud.com/)  
  
I've added a _todo_item_ type which stores the item's value and two other attributes we won't use until the next post when we have user accounts for their own _todo_item_ stores.  
<pre style="color:#bebebe;background-color:#262626;"><span style="color:#00ffff;">type</span> <span style="color:#b0c4de;">todo_item =</span> <span style="color:#00ffff;">{</span> user_id <span style="color:#98fb98;">: string</span>  
                 <span style="color:#00ffff;">;</span> value <span style="color:#98fb98;">: string</span>  
                 <span style="color:#00ffff;">;</span> created_at <span style="color:#98fb98;">: string</span>  
                 <span style="color:#00ffff;">}</span></pre>  
To tell Opa where to store the records we'll create, we provide a path to the Opa db function and set its type. For our todo items we use a _stringmap_ since currently the id's are randomly generated strings (I know, I know, but its just an example!). We can then reference a record in the database with the path /todo_item[some_id_string].  
<pre style="color:#bebebe;background-color:#262626;"><span style="color:#00ffff;">db</span> /todo_items <span style="color:#98fb98;">: stringmap(todo_item)</span></pre>  
Now we can insert _todo_item_'s to this db path as so:  
<pre style="color:#bebebe;background-color:#262626;">/todo_items[id] <span style="color:#00ffff;">&lt;-</span> <span style="color:#00ffff;">{</span> <span style="color:#b0c4de;">value=</span>x <span style="color:#b0c4de;">user_id=</span><span style="color:#ffa07a;">""</span> <span style="color:#b0c4de;">created_at=</span><span style="color:#ffa07a;">""</span> <span style="color:#00ffff;">}</span></pre>  
For now _user_id_ and _created_at_ are empty, but I'll be updating that when I add user accounts.  
  
Since we are storing each item, we need to populate the list on page load with whats already stored:  
<pre style="color:#bebebe;background-color:#262626;"><span style="color:#87cefa;">add_todos</span>() =  
  <span style="color:#b0c4de;">items =</span> /todo_items  
  <span style="color:#87cefa;">StringMap.iter</span>((x, y <span style="color:#00ffff;">-&gt;</span> <span style="color:#87cefa;">add_todo_to_page</span>(x, y<span style="color:#7fffd4;">.value</span>)), items)</pre>  
The first line of the function sets the variable items to all the _todo_item_ records in the database. We use _StringMap.iter_ to take each _todo_item_ and add it to the page. The first argument to the anonymous function is the id the item is stored in the database with (the id we will use in the HTML as well) and the second is the actual _todo_item_, so we take its value field and pass that to the _add_todo_to_page_ function along with the id.  
  
To have the _add_todos_ function when the list element is ready we add an _on_ready_ attribute that will call add_todos:  
<pre style="color:#bebebe;background-color:#262626;"><span style="color:#ffa07a;">&lt;</span><span style="color:#00ffff;">ul </span><span style="color:#ffa07a;">id=#todo_list onready={</span>_ <span style="color:#00ffff;">-&gt;</span> <span style="color:#87cefa;">add_todos</span>() <span style="color:#ffa07a;">} &gt;&lt;</span><span style="color:#00ffff;">/ul</span><span style="color:#ffa07a;">&gt;</span></pre>  
Lastly, we want to be able to delete a _todo_item_ from the database:  
<pre style="color:#bebebe;background-color:#262626;"><span style="color:#87cefa;">remove_item</span>(id<span style="color:#98fb98;">: string</span>) =  
  <span style="color:#00ffff;">do</span> <span style="color:#87cefa;">Dom.remove</span>(<span style="color:#87cefa;">Dom.select_parent_one</span>(#<span style="color:#00ffff;">{</span>id<span style="color:#00ffff;">}</span>))  
  <span style="color:#00ffff;">do</span> <span style="color:#87cefa;">Db.remove</span>(@/todo_items[id])  
  <span style="color:#87cefa;">update_counts</span>()  
  
<span style="color:#87cefa;">remove_all_done</span>() =  
  <span style="color:#87cefa;">Dom.iter</span>(x <span style="color:#00ffff;">-&gt;</span> <span style="color:#87cefa;">remove_item</span>(<span style="color:#87cefa;">Dom.get_id</span>(x)), <span style="color:#87cefa;">Dom.select_class</span>(<span style="color:#ffa07a;">"done"</span>))</pre>  
The main piece to notice here is _@/todo_items[id]_ in _Db.remove()_. The @ is saying that we are passing the path itself to _remove()_ and not the value at that path.  
  
Nice and easy! No database to setup or deploy, just Opa. Next time we'll add user accounts, so we don't have to all share the same todo list.

