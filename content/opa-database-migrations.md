+++
author = "Tristan Sloughter"
type="post"
categories = ["Functional Programming", "Opa"]
date = 2011-10-15T19:11:58Z
description = ""
draft = false
slug = "opa-database-migrations"
tags = ["Functional Programming", "Opa"]
title = "Opa Database Migrations"

+++

[Nicolas Glondu](https://github.com/HenJi) posted a [comment](http://bit.ly/mTu3QV) on an [earlier post](http://blog.erlware.org/2011/10/06/opado-data-storage/) detailing ways for doing database migrations with [Opa](http://opalang.org "Opa"). I thought it was useful enough that I should put up a post around it:  
  
> If you have complex changes in database structures in a OPA program, you have two choices :>   
>   
> 1 – Keep both structure in your database and create a function which populates the new empty field from the other fields. Launch your program once with –db-force-upgrade and launch this function once (adding fields is safe but the new fields are empty). After this, clean removed fields and the function and re-run your program with –db-force-upgrade once more (removing fields is also safe). After this, you can keep using your program normally. A bit long but convenient for changes in small applications (with one instance running). You don’t have to relaunch your program immediately and you can spread ou migration on two program updates.>   
>   
> 2 – Create another OPA program with both database definitions but with root nodes named differently [1]. Your program should take data from the original node and put it in the new node with your changes. That way you can have a standalone opa application for your migration. Which is more convenient if your program is used by other users. Once your program compiled, you can use command line options (–db-local) to use your true program’s database as the input root node an to put the output root node in a safe place. However, I don’t know if it works if your program has no specific root node defined.>   
>   
> [1] You can define a root node with `database my_root = @meta` and use it with `db /my_root/toto : stringmap(int)`. You can have multiple root nodes in one program.

