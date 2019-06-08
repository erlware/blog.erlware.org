+++
author = "Tristan Sloughter"
type="post"
categories = ["batman.js", "coffeescript", "Javascript", "js", "knockout.js", "RESTful"]
date = 2011-08-29T01:37:13Z
description = ""
draft = false
slug = "batman-js-vs-knockout-js"
tags = ["batman.js", "coffeescript", "Javascript", "js", "knockout.js", "RESTful"]
title = "Batman.js vs Knockout.js"

+++

The following is NOT a tutorial for either Batman.js or Knockout.js. But, it is instead a sort of side-by-side comparison of the two for creating a user creation form that POSTs the new user's data as JSON to the backend.  
  
The method of web development I've come to find the best is based on heavy frontend Javascript (though written in Coffeescript) communicating with a backend via a RESTful interface. This is appealing, because you are not cluttering the application logic with view related code. This allows me to use Erlang, my language of choice, which is great at implementing RESTful interfaces with Webmachine, but not too great for trying to build a site like you would with Rails.  
  
I recently began using Knockout.js and found it to be a great fit for my development paradigm. When Batman.js came out, I saw that it could provide me with what Knockout.js does but also already take care of pieces I would develop myself on the frontend, like RESTful persistence.  
  
First, we have the Knockout.js logic, written in Coffeescript, for setting up a User class and object that observers the input fields.  
<pre style="color:#00ff00;background-color:#000000;"><span style="color:#00ffff;">class</span> <span style="color:#eedd82;">@User</span>  
  <span style="color:#98fb98;">constructor :</span> -&gt;  
    <span style="color:#98fb98;">firstname :</span> ko.observable <span style="color:#ffa07a;">""</span>  
    <span style="color:#98fb98;">lastname :</span> ko.observable <span style="color:#ffa07a;">""</span>  
    <span style="color:#98fb98;">email :</span> ko.observable <span style="color:#ffa07a;">""</span>  
    <span style="color:#98fb98;">username :</span> ko.observable <span style="color:#ffa07a;">""</span>  
    <span style="color:#98fb98;">password :</span> ko.observable <span style="color:#ffa07a;">""</span>  
  
  <span style="color:#98fb98;">save :</span> -&gt;  
    <span style="color:#00ffff;">if</span> $(<span style="color:#ffa07a;">"form"</span>).validate().form()  
      $.post(<span style="color:#ffa07a;">'/user'</span>, ko.toJSON(<span style="color:#eedd82;">this</span>), (data) -&gt; window.location = <span style="color:#ffa07a;">"/login.html"</span>; <span style="color:#00ffff;">return</span> <span style="color:#7fffd4;">false</span>;).error () -&gt; alert(<span style="color:#ffa07a;">"error"</span>); <span style="color:#00ffff;">return</span> <span style="color:#7fffd4;">false</span>;  
  
user = <span style="color:#00ffff;">new</span> User  
ko.applyBindings user</pre>  
Now for the HTML for displaying the fields, configuring the submit handler, binding the inputs to Knockout.js observables and setting validators.  
<pre style="color:#00ff00;background-color:#000000;">&lt;<span style="color:#87cefa;">form</span> <span style="color:#eedd82;">data-bind</span>=<span style="color:#ffa07a;">"submit: save"</span>&gt;  
    &lt;<span style="color:#87cefa;">div</span> <span style="color:#eedd82;">class</span>=<span style="color:#ffa07a;">"new_user_box"</span>&gt;  
    &lt;<span style="color:#87cefa;">label</span>&gt;First Name: &lt;/<span style="color:#87cefa;">label</span>&gt;  
    &lt;<span style="color:#87cefa;">input</span> type=text name=firstname <span style="color:#eedd82;">data-bind</span>=<span style="color:#ffa07a;">"value: firstname"</span> minlength=2 maxlength=25 <span style="color:#eedd82;">class</span>=<span style="color:#ffa07a;">"required"</span> /&gt;  
  
    &lt;<span style="color:#87cefa;">label</span>&gt;Last Name: &lt;/<span style="color:#87cefa;">label</span>&gt;  
    &lt;<span style="color:#87cefa;">input</span> type=text name=lastname <span style="color:#eedd82;">data-bind</span>=<span style="color:#ffa07a;">"value: lastname"</span>  minlength=2 maxlength=25 <span style="color:#eedd82;">class</span>=<span style="color:#ffa07a;">"required"</span> /&gt;  
  
    &lt;<span style="color:#87cefa;">label</span>&gt;Username: &lt;/<span style="color:#87cefa;">label</span>&gt;  
    &lt;<span style="color:#87cefa;">input</span> type=text name=username <span style="color:#eedd82;">data-bind</span>=<span style="color:#ffa07a;">"value: username"</span> <span style="color:#eedd82;">remote</span>=<span style="color:#ffa07a;">"/user/check"</span>  minlength=6 maxlength=25 <span style="color:#eedd82;">class</span>=<span style="color:#ffa07a;">"required"</span> /&gt;  
  
    &lt;<span style="color:#87cefa;">label</span>&gt;Password: &lt;/<span style="color:#87cefa;">label</span>&gt;  
    &lt;<span style="color:#87cefa;">input</span> type=password <span style="color:#eedd82;">id</span>=<span style="color:#ffa07a;">"password1"</span> <span style="color:#eedd82;">data-bind</span>=<span style="color:#ffa07a;">"value: password, uniqueName: true"</span> minlength=8 <span style="color:#eedd82;">class</span>=<span style="color:#ffa07a;">"required password"</span> /&gt;  
  
    &lt;<span style="color:#87cefa;">label</span>&gt;Retype Password: &lt;/<span style="color:#87cefa;">label</span>&gt;  
    &lt;<span style="color:#87cefa;">input</span> type=password <span style="color:#eedd82;">data-bind</span>=<span style="color:#ffa07a;">"uniqueName: true"</span>  <span style="color:#eedd82;">id</span>=<span style="color:#ffa07a;">"password1"</span> <span style="color:#eedd82;">equalto</span>=<span style="color:#ffa07a;">"#password1"</span> <span style="color:#eedd82;">class</span>=<span style="color:#ffa07a;">"required"</span> /&gt;  
  
    &lt;<span style="color:#87cefa;">label</span>&gt;Email Address: &lt;/<span style="color:#87cefa;">label</span>&gt;  
    &lt;<span style="color:#87cefa;">input</span> type=email name=email <span style="color:#eedd82;">data-bind</span>=<span style="color:#ffa07a;">"value: email"</span>  <span style="color:#eedd82;">remote</span>=<span style="color:#ffa07a;">"/user/email_check"</span>  <span style="color:#eedd82;">class</span>=<span style="color:#ffa07a;">"required email"</span> /&gt;  
  
    &lt;<span style="color:#87cefa;">input</span> type=submit <span style="color:#eedd82;">value</span>=<span style="color:#ffa07a;">"Save"</span> <span style="color:#eedd82;">id</span>=<span style="color:#ffa07a;">"saveSubmit"</span> /&gt;  
    &lt;/<span style="color:#87cefa;">div</span>&gt;  
&lt;/<span style="color:#87cefa;">form</span>&gt;</pre>  
With Batman.js our validation will be configured in the user model. Here we only need to set @persist to Batman.RestStorage and when a model is saved with the save method it will POST the encoded fields as JSON to /users.  
<pre style="color:#00ff00;background-color:#000000;"><span style="color:#00ffff;">class</span> CT <span style="color:#00ffff;">extends</span> Batman.App  
    <span style="color:#eedd82;">@global</span> <span style="color:#7fffd4;">yes</span>  
    <span style="color:#eedd82;">@root</span> <span style="color:#ffa07a;">'users#index'</span>  
  
<span style="color:#00ffff;">class</span> CT.User <span style="color:#00ffff;">extends</span> Batman.Model  
    <span style="color:#eedd82;">@global</span> <span style="color:#7fffd4;">yes</span>  
    <span style="color:#eedd82;">@persist</span> Batman.RestStorage  
    <span style="color:#eedd82;">@encode</span> <span style="color:#ffa07a;">'firstname'</span>, <span style="color:#ffa07a;">'lastname'</span>, <span style="color:#ffa07a;">'username'</span>, <span style="color:#ffa07a;">'email'</span>, <span style="color:#ffa07a;">'password'</span>  
    <span style="color:#eedd82;">@validate</span> <span style="color:#ffa07a;">'firstname'</span>, <span style="color:#98fb98;">presence:</span> <span style="color:#7fffd4;">yes</span>, <span style="color:#98fb98;">maxLength:</span> 255  
    <span style="color:#eedd82;">@validate</span> <span style="color:#ffa07a;">'lastname'</span>, <span style="color:#98fb98;">presence:</span> <span style="color:#7fffd4;">yes</span>, <span style="color:#98fb98;">maxLength:</span> 255  
    <span style="color:#eedd82;">@validate</span> <span style="color:#ffa07a;">'username'</span>, <span style="color:#98fb98;">presence:</span> <span style="color:#7fffd4;">yes</span>, <span style="color:#98fb98;">lengthWithin:</span> [6,255]  
    <span style="color:#eedd82;">@validate</span> <span style="color:#ffa07a;">'email'</span>, <span style="color:#98fb98;">presence:</span> <span style="color:#7fffd4;">yes</span>  
    <span style="color:#eedd82;">@validate</span> <span style="color:#ffa07a;">'password'</span>, <span style="color:#ffa07a;">'passwordConfirmation'</span>, <span style="color:#98fb98;">presence:</span> <span style="color:#7fffd4;">yes</span>, <span style="color:#98fb98;">lengthWithin:</span> [6,255]  
  
<span style="color:#00ffff;">class</span> CT.UsersController <span style="color:#00ffff;">extends</span> Batman.Controller  
    <span style="color:#98fb98;">user:</span> <span style="color:#7fffd4;">null</span>  
  
    <span style="color:#98fb98;">index:</span> -&gt;  
        <span style="color:#eedd82;">@set</span> <span style="color:#ffa07a;">'user'</span>, <span style="color:#00ffff;">new</span> User  
        <span style="color:#00ffff;">return</span> <span style="color:#7fffd4;">false</span>  
  
    <span style="color:#98fb98;">create:</span> =&gt;  
        <span style="color:#eedd82;">@user</span>.save()  
        <span style="color:#00ffff;">return</span> <span style="color:#7fffd4;">false</span>  
  
CT.run()</pre>  
We see below that the Batman.js HTML is a bit cleaner than that in the Knockout.js example above.  
<pre style="color:#00ff00;background-color:#000000;">        &lt;<span style="color:#87cefa;">form</span> <span style="color:#eedd82;">data-formfor-user</span>=<span style="color:#ffa07a;">"controllers.users.user"</span> <span style="color:#eedd82;">data-event-submit</span>=<span style="color:#ffa07a;">"controllers.users.create"</span>&gt;  
          &lt;<span style="color:#87cefa;">div</span> <span style="color:#eedd82;">class</span>=<span style="color:#ffa07a;">"new_user_box"</span>&gt;  
          &lt;<span style="color:#87cefa;">label</span>&gt;First Name: &lt;/<span style="color:#87cefa;">label</span>&gt;  
          &lt;<span style="color:#87cefa;">input</span> type=text name=firstname <span style="color:#eedd82;">data-bind</span>=<span style="color:#ffa07a;">"user.firstname"</span> /&gt;  
  
          &lt;<span style="color:#87cefa;">label</span>&gt;Last Name: &lt;/<span style="color:#87cefa;">label</span>&gt;  
          &lt;<span style="color:#87cefa;">input</span> type=text name=lastname <span style="color:#eedd82;">data-bind</span>=<span style="color:#ffa07a;">"user.lastname"</span> /&gt;  
  
          &lt;<span style="color:#87cefa;">label</span>&gt;Username: &lt;/<span style="color:#87cefa;">label</span>&gt;  
          &lt;<span style="color:#87cefa;">input</span> type=text name=username <span style="color:#eedd82;">data-bind</span>=<span style="color:#ffa07a;">"user.username"</span> <span style="color:#eedd82;">remote</span>=<span style="color:#ffa07a;">"/user/check"</span> /&gt;  
  
          &lt;<span style="color:#87cefa;">label</span>&gt;Password: &lt;/<span style="color:#87cefa;">label</span>&gt;  
          &lt;<span style="color:#87cefa;">input</span> type=password <span style="color:#eedd82;">data-bind</span>=<span style="color:#ffa07a;">"user.password"</span> /&gt;  
  
          &lt;<span style="color:#87cefa;">label</span>&gt;Retype Password: &lt;/<span style="color:#87cefa;">label</span>&gt;  
          &lt;<span style="color:#87cefa;">input</span> type=password <span style="color:#eedd82;">data-bind</span>=<span style="color:#ffa07a;">"user.passwordConfirmation"</span> /&gt;  
  
          &lt;<span style="color:#87cefa;">label</span>&gt;Email Address: &lt;/<span style="color:#87cefa;">label</span>&gt;  
          &lt;<span style="color:#87cefa;">input</span> type=email <span style="color:#eedd82;">data-bind</span>=<span style="color:#ffa07a;">"user.email"</span> <span style="color:#eedd82;">remote</span>=<span style="color:#ffa07a;">"/user/email_check"</span> /&gt;  
  
          &lt;<span style="color:#87cefa;">input</span> type=submit <span style="color:#eedd82;">value</span>=<span style="color:#ffa07a;">"Save"</span> <span style="color:#eedd82;">id</span>=<span style="color:#ffa07a;">"saveSubmit"</span> /&gt;  
          &lt;/<span style="color:#87cefa;">div</span>&gt;  
  
        &lt;/<span style="color:#87cefa;">form</span>&gt;</pre>

