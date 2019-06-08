+++
author = "Tristan Sloughter"
categories = ["dotcloud", "Functional Programming", "OCaml", "Opa", "Web"]
date = 2011-10-15T17:59:01Z
description = ""
draft = false
slug = "opado-personal-todo-lists"
tags = ["dotcloud", "Functional Programming", "OCaml", "Opa", "Web"]
title = "OpaDo: Personal ToDo Lists"

+++

This is a continuation of two past posts ([one](http://blog.erlware.org/2011/10/04/todomvc-in-opa/ "TodoMVC in Opa"), [two](http://blog.erlware.org/2011/10/06/opado-data-storage/ "OpaDo Data Storage")) on my first application with [Opa](http://opalang.com "Opa") called OpaDo. You can try the [live demo here](http://www.opado.org "OpaDo Demo") and check out the full source code on [Github](https://github.com/tsloughter/opado "OpaDo Source")  
  
Updating OpaDo to add user accounts the project structure has been changed a bit and modularized. Below is the new project layout.  
<pre style="color:#00ff00;background-color:#000000;">opado/  
├── Makefile  
├── README.md  
├── dotcloud.yml  
├── resources  
│   ├── destroy.png  
│   └── todos.css  
└── src  
    ├── main.opa  
    ├── todo.opa  
    └── user.opa</pre>  
Now there is a _main_, _todo_ and _user_ module. The _main_ module is the entry point for the app and looks like:  
<pre style="color:#00ff00;background-color:#000000;"><span style="color:#00ffff;">package</span> opado<span style="color:#7fffd4;">.main</span>  
  
<span style="color:#00ffff;">import</span> opado<span style="color:#7fffd4;">.user</span>  
<span style="color:#00ffff;">import</span> opado<span style="color:#7fffd4;">.todo</span>  
  
<span style="color:#b0c4de;">urls </span><span style="color:#98fb98;">: Parser.general_parser(http_request -&gt; resource)</span><span style="color:#b0c4de;"> =</span>  
  <span style="color:#00ffff;">parser</span>  
  | <span style="color:#00ffff;">{</span><span style="color:#87cefa;">Rule.debug_parse_string</span>(s <span style="color:#00ffff;">-&gt;</span> <span style="color:#87cefa;">Log.notice</span>(<span style="color:#ffa07a;">"URL"</span>,s))<span style="color:#00ffff;">}</span> Rule<span style="color:#7fffd4;">.fail</span> <span style="color:#00ffff;">-&gt;</span> <span style="color:#87cefa;">error</span>(<span style="color:#ffa07a;">""</span>)  
  | <span style="color:#ffa07a;">"/todos"</span> <span style="color:#b0c4de;">result=</span><span style="color:#00ffff;">{</span>Todo<span style="color:#7fffd4;">.resource</span><span style="color:#00ffff;">}</span> <span style="color:#00ffff;">-&gt;</span> result  
  | <span style="color:#ffa07a;">"/user"</span> <span style="color:#b0c4de;">result=</span><span style="color:#00ffff;">{</span>User<span style="color:#7fffd4;">.resource</span><span style="color:#00ffff;">}</span> <span style="color:#00ffff;">-&gt;</span> result  
  | <span style="color:#ffa07a;">"/login"</span> <span style="color:#b0c4de;">result=</span><span style="color:#00ffff;">{</span>User<span style="color:#7fffd4;">.resource</span><span style="color:#00ffff;">}</span> <span style="color:#00ffff;">-&gt;</span> result  
  | (.<span style="color:#00ffff;">*</span>) <span style="color:#b0c4de;">result=</span><span style="color:#00ffff;">{</span>Todo<span style="color:#7fffd4;">.resource</span><span style="color:#00ffff;">}</span> <span style="color:#00ffff;">-&gt;</span> result  
  
<span style="color:#00ffff;">server</span><span style="color:#b0c4de;"> =</span> <span style="color:#87cefa;">Server.of_bundle</span>([<span style="color:#cd3700;">@static_resource_directory</span>(<span style="color:#ffa07a;">"resources"</span>)])  
<span style="color:#00ffff;">server</span><span style="color:#b0c4de;"> =</span> <span style="color:#87cefa;">Server.make</span>(urls)</pre>  
Here we define the name of this package and import the _user_ and _todo_ modules. Next is the url matching code. _urls_ is a parser that takes an HTTP request and returns a resource. The matching is pretty straight forward. For example:  
<pre style="color:#00ff00;background-color:#000000;">  | <span style="color:#ffa07a;">"/todos"</span> <span style="color:#b0c4de;">result=</span><span style="color:#00ffff;">{</span>Todo<span style="color:#7fffd4;">.resource</span><span style="color:#00ffff;">}</span> <span style="color:#00ffff;">-&gt;</span> result</pre>  
Here we are matching on URLs that begin with _/todos_ but could have anything after that. What is contained after _/todos_ is passed to the _Todo.resource_ which the variable result is set to. And finally that result is returned.  
  
The last two lines simple define the reource directory for the server and pass in the matching function for the HTTP requests.  
  
The _todo_ resource isn't important to us in this post since its hardly changed. But there are a two important changes:  
<pre style="color:#00ff00;background-color:#000000;"><span style="color:#00ffff;">db</span> /todo_items <span style="color:#98fb98;">: stringmap(stringmap(todo_item))</span>  
<span style="color:#00ffff;">db</span> /todo_items[_][_]/<span style="color:#b0c4de;">done =</span> <span style="color:#00ffff;">false</span></pre>  
Here we see that the _/todo_items_ database is not longer simply a _stringmap_ of _todo_item_'s but a _stringmap_ of a that. This is so we can reference the items by a user identifier. For example a user identified by the string "user01" who has a todo item identified by "aaa" would be read from the data base as _/todo["user01"]["aaa"]_.  
  
There are a few other changes to the todo module so that items are properly inserted for the logged in user and deleting must be done in the second _stringmap_. But we'll move on to the _user_ module now.  
  
Much of the user module was taken from [Matthieu Guffroy's OpaCMS code on github](https://github.com/mattgu74/OpaCms). But I've made a number of modification for my needs.  
<pre style="color:#00ff00;background-color:#000000;"><span style="color:#cd3700;">@abstract</span> <span style="color:#00ffff;">type</span> <span style="color:#b0c4de;">User.password =</span> string  
<span style="color:#cd3700;">@abstract</span> <span style="color:#00ffff;">type</span> <span style="color:#b0c4de;">User.ref =</span> string  
  
<span style="color:#00ffff;">type</span> <span style="color:#b0c4de;">User.t =</span>  
  <span style="color:#00ffff;">{</span>  
    username <span style="color:#98fb98;">: string</span>  
    fullname <span style="color:#98fb98;">: string</span>  
    password <span style="color:#98fb98;">: User.password</span>  
  <span style="color:#00ffff;">}</span>  
  
<span style="color:#00ffff;">type</span> <span style="color:#b0c4de;">User.status =</span> <span style="color:#00ffff;">{</span> logged <span style="color:#98fb98;">: User.ref</span> <span style="color:#00ffff;">}</span> / <span style="color:#00ffff;">{</span> unlogged <span style="color:#00ffff;">}</span>  
<span style="color:#00ffff;">type</span> <span style="color:#b0c4de;">User.info =</span> <span style="color:#87cefa;">UserContext.t</span>(User<span style="color:#7fffd4;">.status</span>)  
<span style="color:#00ffff;">type</span> <span style="color:#87cefa;">User.map</span>('a) = <span style="color:#87cefa;">ordered_map</span>(User<span style="color:#7fffd4;">.ref</span>, 'a, String<span style="color:#7fffd4;">.order</span>)  
  
<span style="color:#00ffff;">db</span> /users <span style="color:#98fb98;">: User.map(User.t)</span>  
  
<span style="color:#b0c4de;">User_data =</span> <span style="color:#00ffff;">\{\{</span>  
  <span style="color:#87cefa;">mk_ref</span>( login <span style="color:#98fb98;">: string</span> ) <span style="color:#98fb98;">: User.ref</span><span style="color:#b0c4de;"> =</span>  
    <span style="color:#87cefa;">String.to_lower</span>(login)  
  
  <span style="color:#87cefa;">ref_to_string</span>( login <span style="color:#98fb98;">: User.ref</span> ) <span style="color:#98fb98;">: string</span><span style="color:#b0c4de;"> =</span>  
    login  
  
  <span style="color:#87cefa;">save</span>( ref <span style="color:#98fb98;">: User.ref</span>, user <span style="color:#98fb98;">: User.t</span> ) <span style="color:#98fb98;">: void</span><span style="color:#b0c4de;"> =</span>  
    /users[ref] <span style="color:#00ffff;">&lt;-</span> user  
  
  <span style="color:#87cefa;">get</span>( ref <span style="color:#98fb98;">: User.ref</span> ) <span style="color:#98fb98;">: option(User.t)</span><span style="color:#b0c4de;"> =</span>  
    ?/users[ref]  
<span style="color:#00ffff;">}}</span></pre>  
Above we have the data, types and database definitions necessary to handle the users.  
  
_User.t_ provides the record for storing necessary user data. Next, we have types for checking the user status of if they are logged in or not.  
  
_UserContext_ is a module provided by Opa for dealing with associating the user values with the client -- via cookies. And the data for that user can only accessed by the user that owns it.  
  
_User_data_ object provides functions for accessing and manipulating users.  
  
Now we can look at the _User_ module.  
<pre style="color:#00ff00;background-color:#000000;"><span style="color:#b0c4de;">User =</span> <span style="color:#00ffff;">\{\{</span>  
  
  <span style="color:#cd3700;">@private</span> <span style="color:#b0c4de;">state =</span> <span style="color:#87cefa;">UserContext.make</span>(<span style="color:#00ffff;">{</span> unlogged <span style="color:#00ffff;">}</span> <span style="color:#98fb98;">: User.status</span>)  
  
  <span style="color:#87cefa;">create</span>(username, password) =  
    <span style="color:#00ffff;">do</span> <span style="color:#00ffff;">match</span> ?/users[username] <span style="color:#00ffff;">with</span>  
      | <span style="color:#00ffff;">{</span>none<span style="color:#00ffff;">}</span> <span style="color:#00ffff;">-&gt;</span>  
          <span style="color:#b0c4de;">user </span><span style="color:#98fb98;">: User.t</span><span style="color:#b0c4de;"> =</span>  
            <span style="color:#00ffff;">{</span> <span style="color:#b0c4de;">username=</span>username <span style="color:#00ffff;">;</span>  
              <span style="color:#b0c4de;">fullname=</span><span style="color:#ffa07a;">""</span> <span style="color:#00ffff;">;</span>  
              <span style="color:#b0c4de;">password =</span> <span style="color:#87cefa;">Crypto.Hash.sha2</span>(password) <span style="color:#00ffff;">}</span>  
          /users[username] <span style="color:#00ffff;">&lt;-</span> user  
  
      | _ <span style="color:#00ffff;">-&gt;</span> void  
    <span style="color:#87cefa;">Client.goto</span>(<span style="color:#ffa07a;">"/login"</span>)</pre>  
At the beginning of the _User_ object we declare a _UserContext_ and a function for creating new users. The function simply checks if the user exists already with the match statement and if not creates a new _User.t_ record and inserts it to the users database.  
  
If we wish to login we must also modify the _UserContext_  
<pre style="color:#00ff00;background-color:#000000;">  <span style="color:#87cefa;">login</span>(login, password) =  
    <span style="color:#b0c4de;">useref =</span> <span style="color:#87cefa;">User_data.mk_ref</span>(login)  
    <span style="color:#b0c4de;">user =</span> <span style="color:#87cefa;">User_data.get</span>(useref)  
    <span style="color:#00ffff;">do</span> <span style="color:#00ffff;">match</span> user <span style="color:#00ffff;">with</span>  
     | <span style="color:#00ffff;">{</span><span style="color:#b0c4de;">some =</span> u<span style="color:#00ffff;">}</span> <span style="color:#00ffff;">-&gt;</span> <span style="color:#00ffff;">if</span> u<span style="color:#7fffd4;">.password</span> == <span style="color:#87cefa;">Crypto.Hash.sha2</span>(password) <span style="color:#00ffff;">then</span>  
                       <span style="color:#87cefa;">UserContext.change</span>(( _ <span style="color:#00ffff;">-&gt;</span> <span style="color:#00ffff;">{</span> <span style="color:#b0c4de;">logged =</span> <span style="color:#87cefa;">User_data.mk_ref</span>(login) <span style="color:#00ffff;">}</span>), state)  
     | _ <span style="color:#00ffff;">-&gt;</span> void  
    <span style="color:#87cefa;">Client.goto</span>(<span style="color:#ffa07a;">"/todos"</span>)</pre>  
The function attempts to read the user from the database and checks if the passwords match. If so, it will set the _UserContext_ to logged in. The function then tells the client to go to _/todos_. If the login was unsuccessful, it doesn't matter and will just redirect to the sign up page.  
  
Obviously, better error handling and notification is the next step for the application.  
  
The last interesting part for this I think is the request matching. The rest of the code is mostly just HTML and piecing together the functions I already described.  
<pre style="color:#00ff00;background-color:#000000;">  <span style="color:#b0c4de;">resource </span><span style="color:#98fb98;">: Parser.general_parser(http_request -&gt; resource)</span><span style="color:#b0c4de;"> =</span>  
    <span style="color:#00ffff;">parser</span>  
    | <span style="color:#ffa07a;">"/new"</span> <span style="color:#00ffff;">-&gt;</span>  
      _req <span style="color:#00ffff;">-&gt;</span> <span style="color:#87cefa;">Resource.styled_page</span>(<span style="color:#ffa07a;">"New User"</span>, [<span style="color:#ffa07a;">"/resources/todos.css"</span>], <span style="color:#87cefa;">new</span>())  
    | <span style="color:#ffa07a;">"/edit"</span> <span style="color:#00ffff;">-&gt;</span>  
      _req <span style="color:#00ffff;">-&gt;</span> <span style="color:#87cefa;">edit</span>()  
    | <span style="color:#ffa07a;">"/view/"</span> <span style="color:#b0c4de;">login=</span>(.<span style="color:#00ffff;">*</span>) <span style="color:#00ffff;">-&gt;</span>  
      _req <span style="color:#00ffff;">-&gt;</span> <span style="color:#87cefa;">view</span>(<span style="color:#87cefa;">Text.to_string</span>(login))  
    | .<span style="color:#00ffff;">*</span> <span style="color:#00ffff;">-&gt;</span>  
      _req <span style="color:#00ffff;">-&gt;</span> <span style="color:#87cefa;">start</span>()</pre>  
The key match to look at is:  
<pre style="color:#00ff00;background-color:#000000;">    | <span style="color:#ffa07a;">"/view/"</span> <span style="color:#b0c4de;">login=</span>(.<span style="color:#00ffff;">*</span>) <span style="color:#00ffff;">-&gt;</span>  
      _req <span style="color:#00ffff;">-&gt;</span> <span style="color:#87cefa;">view</span>(<span style="color:#87cefa;">Text.to_string</span>(login))</pre>  
  
This shows the request matching _/view_, which in this case comes after the main module matches '/user' and routes to the User module resource. But then we have _login=(.*)_, this is matching the variable login to the rest of the url. This variable _login_ can then be used in _view(Text.to_string(login))_ to pass to the _view_ function so it knows what user is being asked to be displayed.  
  
There'll be more to come. Next, I need to add some validation, an admin page and then the ability for users to have categories to organize their todo items under.  
  
And let me know anything else people would like to see!

