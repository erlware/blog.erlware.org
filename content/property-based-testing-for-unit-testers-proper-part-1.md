+++
author = "Tristan Sloughter"
type="post"
date = 2011-07-12T18:08:16Z
description = ""
draft = false
slug = "property-based-testing-for-unit-testers-proper-part-1"
title = "Property based testing for unit testers with PropEr - Part 1"

+++

> This tutorial is brought to you by [ErlangCamp 2011 (click here)](http://www.erlangcamp.com) - Boston, August 12th and 13th - It's gonna be totally sweet!  
  
Main contributors: Torben Hoffmann, Raghav Karol, Eric Merritt  
  
The purpose of the short document is to help people who are familiar  
with unit testing understand how property based testing (PBT) differs,  
but also where the thinking is the same.  
  
This document focusses on the PBT tool  
[`PropEr`](https://github.com/manopapad/proper) for Erlang since that is  
what I am familiar with, but the general principles applies to all PBT  
tools regardless of which language they are written in.  
  
The approach taken here is that we hear from people who are used to  
working with unit testing regarding how they think when designing  
their tests and how a concrete test might look.  
  
These descriptions are then "converted" into the way it works with  
PBT, with a clear focus on what stays the same and what is different.  
  
## Testing philosophies  
  
### A quote from Martin Logan ([@martinjlogan](http://twitter.com/martinjlogan)):  
  
> For me unit testing is about contracts. I think about the same things>   
> I think about when I write statements like {ok, Resp} =>   
> Mod:Func(Args). Unit testing and writing specs are very close for me.>   
> Hypothetically speaking lets say a function should return return {ok,>   
> string()} | {error, term()} for all given input parameters then my>   
> unit tests should be able to show that for a representative set of>   
> input parameters that those contracts are honored. The art comes in>   
> thinking about what that set is.  
  
The trap in writing all your own tests can often be that we think  
about the set in terms of what we coded for and not what may indeed be  
asked of our function. As the code is tried in further exploratory  
testing and in production new input parameter sets for which the given  
function does not meet the stated contract are discovered and added to  
the test case once a fix has been put into place.  
  
This is a very good description of what the ground rules for unit  
testing are:  
  
*   Checking that contracts are obeyed.  
*   Creating a representative set of input parameters.  
  
The former is very much part of PBT - each property you write will  
check a contract, so that thinking is the same.  
  
## xUnit vs PBT  
  
Unit testing has become popular for software testing with the advent  
of xUnit tools like jUnit for Java.  xUnit like tools typically  
provide a testing framework with the following functionality  
  
*   test fixture setup  
*   test case execution  
*   test fixture teardown  
*   test suite management  
*   test status reporting and management  
  
While xUnit tools provide a lot of functionality to execute and manage  
test cases and suites, reporting results there is no focus on test  
case execution step, while this is the main focus area of  
property-based testing (PBT).  
  
Consider the following function specification  
  
<div class="codehilite"><pre><span class="nf">sort</span><span class="p">(</span><span class="nn">list</span><span class="p">::</span><span class="n">integer</span><span class="p">())</span> <span class="o">---&gt;</span> <span class="nn">list</span><span class="p">::</span><span class="n">integer</span><span class="p">()</span> <span class="p">|</span> <span class="n">error</span>  
</pre></div>  
  
A verbal specification of this function is,  
  
> For all input lists of integers, the sort function returns a sorted>   
> list of integers.  
  
For any other kind of argument the function returns the atom error.  
  
The specification above may be a requirement of how the function  
should behave or even how the function does behave. This distinction  
is important; the former is the requirement for the function, the  
latter is the actual API. Both should be the same and that is what our  
testing should confirm. Test cases for this function might look like  
  
<div class="codehilite"><pre><span class="nf">assertEqual</span><span class="p">(</span><span class="n">sort</span><span class="p">([</span><span class="mi">5</span><span class="p">,</span><span class="mi">4</span><span class="p">,</span><span class="mi">3</span><span class="p">,</span><span class="mi">2</span><span class="p">,</span><span class="mi">1</span><span class="p">]),</span> <span class="p">[</span><span class="mi">1</span><span class="p">,</span><span class="mi">2</span><span class="p">,</span><span class="mi">3</span><span class="p">,</span><span class="mi">4</span><span class="p">,</span><span class="mi">5</span><span class="p">])</span>  
<span class="nf">assertEqual</span><span class="p">(</span><span class="n">sort</span><span class="p">([</span><span class="mi">1</span><span class="p">,</span><span class="mi">2</span><span class="p">,</span><span class="mi">3</span><span class="p">,</span><span class="mi">4</span><span class="p">,</span><span class="mi">5</span><span class="p">]),</span> <span class="p">[</span><span class="mi">1</span><span class="p">,</span><span class="mi">2</span><span class="p">,</span><span class="mi">3</span><span class="p">,</span><span class="mi">4</span><span class="p">,</span><span class="mi">5</span><span class="p">])</span>  
<span class="nf">assertEqual</span><span class="p">(</span><span class="n">sort</span><span class="p">([]</span>         <span class="p">),</span> <span class="p">[]</span>         <span class="p">)</span>  
<span class="nf">assertEqual</span><span class="p">(</span><span class="n">sort</span><span class="p">([</span><span class="o">-</span><span class="mi">1</span><span class="p">,</span><span class="mi">0</span><span class="p">,</span> <span class="mi">1</span><span class="p">]</span>  <span class="p">),</span> <span class="p">[</span><span class="o">-</span><span class="mi">1</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="mi">1</span><span class="p">]</span> <span class="p">)</span>  
</pre></div>  
  
How many tests cases should we write to be convinced that the actual  
behaviour of the function is the same as its specification? Clearly,  
it is impossible to write tests cases for all possible input values,  
here all lists of integers, the art of testing is finding individual  
input values that are representative of a large part of the input  
space. We hope that the test cases are exhaustive to cover the  
specification. xUnit tools offer no support for this and this is where  
PBT and PBT Tools like `PropEr` and `QuickCheck` come in.  
  
PBT introduces testing with a large set of random input values and  
verifying that the specification holds for each input value  
selected. Functions used to generate input values, generators, are  
specified using rules and can be simply composed together to construct  
complicated values.  So, a property based test for the function above  
may look like:  
  
<div class="codehilite"><pre><span class="nv">FOREACH</span><span class="p">({</span><span class="nv">I</span><span class="p">,</span> <span class="nv">J</span><span class="p">,</span> <span class="nv">InputList</span><span class="p">},</span>  <span class="p">{</span><span class="n">nat</span><span class="p">(),</span> <span class="n">nat</span><span class="p">(),</span> <span class="n">integer_list</span><span class="p">()},</span>  
    <span class="nv">SUCHTHAT</span><span class="p">(</span><span class="nv">I</span> <span class="o">&lt;</span> <span class="nv">J</span> <span class="ow">andalso</span> <span class="nv">J</span> <span class="o">&lt;</span> <span class="nb">length</span><span class="p">(</span><span class="nv">InputList</span><span class="p">),</span>  
    <span class="nv">SortedList</span> <span class="o">=</span> <span class="n">sort</span><span class="p">(</span><span class="nv">InputList</span><span class="p">)</span>  
    <span class="nb">length</span><span class="p">(</span><span class="nv">SortedList</span><span class="p">)</span> <span class="o">==</span> <span class="nb">length</span><span class="p">(</span><span class="nv">InputList</span><span class="p">)</span>  
    <span class="ow">andalso</span>  
    <span class="nn">lists</span><span class="p">:</span><span class="nb">get</span><span class="p">(</span><span class="nv">SortedList</span><span class="p">,</span> <span class="nv">I</span><span class="p">)</span> <span class="o">=&lt;</span> <span class="nn">lists</span><span class="p">:</span><span class="nb">get</span><span class="p">(</span><span class="nv">SortedList</span><span class="p">,</span> <span class="nv">J</span><span class="p">))</span>  
</pre></div>  
  
The property above works as follows  
  
*   Generate a random list of integers `InputList` and two natural numbers  
  I, J, such that I &lt; J &lt; size of `InputList`  
*   Check that size of sorted and input lists is the same.  
*   Check that element with smaller index I is less than or equal to  
  element with larger index J in `SortedList`.  
  
Notice in the property above, we _specify_ property. Verification of  
the property based on random input values will be done by the property  
based tool, therefore we can generated a large number of tests cases  
with random input values and have a higher level of confidence that  
the function when using unit tests alone.  
  
But it does not stop at generation of input parameters. If you have  
more complex tests where you have to generate a series of events and  
keep track of some state then your PBT tool will generate random  
sequences of events which corresponds to legal sequences of events and  
test that your system behaves correctly for all sequences.  
  
So when you have written a property with associated generators you  
have in fact created something that can create numerous test cases -  
you just have to tell your PBT tool how many test cases you want to  
check the property on.  
  
## Shrinking the bar  
  
At this point you might still have the feeling that introducing the  
notion of some sort of generators to your unit testing tool of choice  
would bring you on par with PBT tools, but wait there is more to  
come.  
  
When a PBT tool creates a test case that fails there is real chance  
that it has created a long test case or some big input parameters -  
trying to debug that is very much like receiving a humongous log from  
a system in the field and try to figure out what cause the system to  
fail.  
  
Enter shrinking...  
  
When a test case fails the PBT tool will try to shrink the failing  
test case down to the essentials by stripping out input elements or  
events that does not cause the failure. In most cases this results in  
a very short counterexample that clearly states which events and  
inputs are required to break a property.  
  
As we go through some concrete examples later the effects of shrinking  
will be shown.  
  
Shrinking makes it a lot easier to debug problems and is as key to the  
strength of PBT as the generators.  
  
## Converting a unit test  
  
We will now take a look at one possible way of translating a unit  
test into a PBT setting.  
  
The example comes from Eric Merritt and is about the `add/2` function in  
the `ec_dictionary` instance `ec_gb_trees`.  
  
The add function has the following spec:  
  
<div class="codehilite"><pre><span class="p">-</span><span class="ni">spec</span> <span class="n">add</span><span class="p">(</span><span class="nn">ec_dictionary</span><span class="p">:</span><span class="n">key</span><span class="p">(),</span> <span class="nn">ec_dictionary</span><span class="p">:</span><span class="n">value</span><span class="p">(),</span> <span class="nv">Object</span><span class="p">::</span><span class="n">dictionary</span><span class="p">())</span> <span class="o">-&gt;</span>  
          <span class="n">dictionary</span><span class="p">().</span>  
</pre></div>  
  
and it is supposed to do the obvious: add the key and value pair to  
the dictionary and return a new dictionary.  
  
Eric states his basic expectations as follows:  
  
1.  I can put arbitrary terms into the dictionary as keys  
2.  I can put arbitrary terms into the dictionary as values  
3.  When I put a value in the dictionary by a key, I can retrieve that same value  
4.  When I put a different value in the dictionary by key it does not change other key value pairs.  
5.  When I update a value the new value in available by the new key  
6.  When a value does not exist a not found exception is created  
  
The first two expectations regarding being able to use arbritrary  
terms as keys and values is a job for generators.  
  
The latter four are prime candidates for properties and we will create  
one for each of them.  
  
### Generators  
  
<div class="codehilite"><pre><span class="nf">key</span><span class="p">()</span> <span class="o">-&gt;</span> <span class="n">any</span><span class="p">().</span>  
  
<span class="nf">value</span><span class="p">()</span> <span class="o">-&gt;</span> <span class="n">any</span><span class="p">().</span>  
</pre></div>  
  
For `PropEr` this approach has the drawback that creation and shrinking  
becomes rather time consuming, so it might be better to narrow to  
something like this:  
  
<div class="codehilite"><pre><span class="nf">key</span><span class="p">()</span> <span class="o">-&gt;</span> <span class="n">union</span><span class="p">([</span><span class="n">integer</span><span class="p">(),</span><span class="n">atom</span><span class="p">()]).</span>  
  
<span class="nf">value</span><span class="p">()</span> <span class="o">-&gt;</span> <span class="n">union</span><span class="p">([</span><span class="n">integer</span><span class="p">(),</span><span class="n">atom</span><span class="p">(),</span><span class="n">binary</span><span class="p">(),</span><span class="n">boolean</span><span class="p">(),</span><span class="n">string</span><span class="p">()]).</span>  
</pre></div>  
  
What is best depends on the situation and intended usage.  
  
Now, being able to generate keys and values is not enough. You also  
have to tell `PropEr` how to create a dictionary and in this case we  
will use a symbolic generator (detail to be explained later).  
  
<div class="codehilite"><pre><span class="nf">sym_dict</span><span class="p">()</span> <span class="o">-&gt;</span>  
    <span class="no">?SIZED</span><span class="p">(</span><span class="nv">N</span><span class="p">,</span><span class="n">sym_dict</span><span class="p">(</span><span class="nv">N</span><span class="p">)).</span>  
  
<span class="nf">sym_dict</span><span class="p">(</span><span class="mi">0</span><span class="p">)</span> <span class="o">-&gt;</span>  
    <span class="p">{</span><span class="n">&#039;$call&#039;</span><span class="p">,</span><span class="n">ec_dictionary</span><span class="p">,</span><span class="n">new</span><span class="p">,[</span><span class="n">ec_gb_trees</span><span class="p">]};</span>  
<span class="nf">sym_dict</span><span class="p">(</span><span class="nv">N</span><span class="p">)</span> <span class="o">-&gt;</span>  
    <span class="no">?LAZY</span><span class="p">(</span>  
       <span class="n">frequency</span><span class="p">([</span>  
                  <span class="p">{</span><span class="mi">1</span><span class="p">,</span> <span class="p">{</span><span class="n">&#039;$call&#039;</span><span class="p">,</span><span class="n">ec_dictionary</span><span class="p">,</span><span class="n">remove</span><span class="p">,[</span><span class="n">key</span><span class="p">(),</span><span class="n">sym_dict</span><span class="p">(</span><span class="nv">N</span><span class="o">-</span><span class="mi">1</span><span class="p">)]}},</span>  
                  <span class="p">{</span><span class="mi">2</span><span class="p">,</span> <span class="p">{</span><span class="n">&#039;$call&#039;</span><span class="p">,</span><span class="n">ec_dictionary</span><span class="p">,</span><span class="n">add</span><span class="p">,[</span><span class="n">value</span><span class="p">(),</span><span class="n">value</span><span class="p">(),</span><span class="n">sym_dict</span><span class="p">(</span><span class="nv">N</span><span class="o">-</span><span class="mi">1</span><span class="p">)]}}</span>  
                 <span class="p">])).</span>  
</pre></div>  
  
`sym_dict/0` uses the `?SIZED` macro to control the size of the  
generated dictionary. `PropEr` will start out with small numbers and  
gradually raise it.  
  
`sym_dict/1` is building a dictionary by randomly adding key/value  
pairs and removing keys. Eventually the base case is reached which  
will create an empty dictionary.  
  
The `?LAZY` macro is used to defer the calculation of the  
`sym_dict(N-1)` until they are needed and `frequency/1` is used  
to ensure that twice as many adds compared to removes are done. This  
should give rather more interesting dictionaries in the long run, if  
not one can alter the frequencies accondingly.  
  
But does it really work?  
  
That is a good question and one that should always be asked when  
looking at genetors. Fortunately there is a way to see what a  
generator produces provided that the generator functions are exported.  
  
Hint: in most cases it will not hurt to throw in a  
`-compile(export_all).` in the module used to specify the  
properties. And here we actually have a sub-hint: specify the  
properties in a separate file to avoid peeking inside the  
implementation! Base the test on the published API as this is what the  
users of the code will be restricted to.  
  
When the test module has been loaded you can test the generators by  
starting up an Erlang shell (this example uses the erlware_commons  
code so get yourself a clone to play with):  
  
<div class="codehilite"><pre><span class="nv">$ </span>erl -pz ebin -pz <span class="nb">test</span>  
1&gt; proper_gen:pick<span class="o">(</span>ec_dictionary_proper:key<span class="o">())</span>.  
<span class="o">{</span>ok,4<span class="o">}</span>  
2&gt; proper_gen:pick<span class="o">(</span>ec_dictionary_proper:key<span class="o">())</span>.  
<span class="o">{</span>ok,35<span class="o">}</span>  
3&gt; proper_gen:pick<span class="o">(</span>ec_dictionary_proper:key<span class="o">())</span>.  
<span class="o">{</span>ok,-5<span class="o">}</span>  
4&gt; proper_gen:pick<span class="o">(</span>ec_dictionary_proper:key<span class="o">())</span>.  
<span class="o">{</span>ok,48<span class="o">}</span>  
5&gt; proper_gen:pick<span class="o">(</span>ec_dictionary_proper:key<span class="o">())</span>.  
<span class="o">{</span>ok,<span class="s1">&#039;36\207_lÃ Â´?\nc&#039;</span><span class="o">}</span>  
6&gt; proper_gen:pick<span class="o">(</span>ec_dictionary_proper:value<span class="o">())</span>.  
<span class="o">{</span>ok,2<span class="o">}</span>  
7&gt; proper_gen:pick<span class="o">(</span>ec_dictionary_proper:value<span class="o">())</span>.  
<span class="o">{</span>ok,-14<span class="o">}</span>  
8&gt; proper_gen:pick<span class="o">(</span>ec_dictionary_proper:value<span class="o">())</span>.  
<span class="o">{</span>ok,-3<span class="o">}</span>  
9&gt; proper_gen:pick<span class="o">(</span>ec_dictionary_proper:value<span class="o">())</span>.  
<span class="o">{</span>ok,27<span class="o">}</span>  
10&gt; proper_gen:pick<span class="o">(</span>ec_dictionary_proper:value<span class="o">())</span>.  
<span class="o">{</span>ok,-8<span class="o">}</span>  
11&gt; proper_gen:pick<span class="o">(</span>ec_dictionary_proper:value<span class="o">())</span>.  
<span class="o">{</span>ok,<span class="o">[</span>472765,17121<span class="o">]}</span>  
12&gt; proper_gen:pick<span class="o">(</span>ec_dictionary_proper:value<span class="o">())</span>.  
<span class="o">{</span>ok,true<span class="o">}</span>  
13&gt; proper_gen:pick<span class="o">(</span>ec_dictionary_proper:value<span class="o">())</span>.  
<span class="o">{</span>ok,&lt;&lt;&gt;&gt;<span class="o">}</span>  
14&gt; proper_gen:pick<span class="o">(</span>ec_dictionary_proper:value<span class="o">())</span>.  
<span class="o">{</span>ok,<span class="s">&lt;&lt;89,69,18</span>,148,32,42,238,101&gt;&gt;<span class="o">}</span>  
15&gt; proper_gen:pick<span class="o">(</span>ec_dictionary_proper:sym_dict<span class="o">())</span>.  
<span class="o">{</span>ok,<span class="o">{</span><span class="s1">&#039;$call&#039;</span>,ec_dictionary,add,  
        <span class="o">[[</span>114776,1053475<span class="o">]</span>,  
         <span class="s1">&#039;fÂª20\227\215&#039;</span>,  
         <span class="o">{</span><span class="s1">&#039;$call&#039;</span>,ec_dictionary,add,  
             <span class="o">[</span><span class="s1">&#039;&#039;</span>,true,  
              <span class="o">{</span><span class="s1">&#039;$call&#039;</span>,ec_dictionary,add,  
                  <span class="o">[</span><span class="s1">&#039;2^Ã˜Â¡&#039;</span>,  
                   <span class="o">[</span>900408,886056<span class="o">]</span>,  
                   <span class="o">{</span><span class="s1">&#039;$call&#039;</span>,ec_dictionary,add,<span class="o">[[</span>48618|...<span class="o">]</span>,&lt;&lt;...&gt;&gt;|...<span class="o">]}]}]}]}}</span>  
16&gt; proper_gen:pick<span class="o">(</span>ec_dictionary_proper:sym_dict<span class="o">())</span>.  
<span class="o">{</span>ok,<span class="o">{</span><span class="s1">&#039;$call&#039;</span>,ec_dictionary,add,  
        <span class="o">[</span>10,<span class="s1">&#039;aÂ¯\21431fÃµC&#039;</span>,  
         <span class="o">{</span><span class="s1">&#039;$call&#039;</span>,ec_dictionary,add,  
             <span class="o">[</span><span class="nb">false</span>,-1,  
              <span class="o">{</span><span class="s1">&#039;$call&#039;</span>,ec_dictionary,remove,  
                  <span class="o">[</span><span class="s1">&#039;dÂ·Ã‰VÃ·[&#039;</span>,  
                   <span class="o">{</span><span class="s1">&#039;$call&#039;</span>,ec_dictionary,remove,<span class="o">[</span>12,<span class="o">{</span><span class="s1">&#039;$call&#039;</span>,...<span class="o">}]}]}]}]}}</span>  
</pre></div>  
  
That does not look too bad, so we will continue with that for now.  
  
### Properties of `add/2`  
  
The first expectation Eric had about how the dictionary works was that  
if a key had been stored it could be retrieved.  
  
One way of expressing this could be with this property:  
  
<div class="codehilite"><pre><span class="nf">prop_get_after_add_returns_correct_value</span><span class="p">()</span> <span class="o">-&gt;</span>  
    <span class="no">?FORALL</span><span class="p">({</span><span class="nv">Dict</span><span class="p">,</span><span class="nv">K</span><span class="p">,</span><span class="nv">V</span><span class="p">},</span> <span class="p">{</span><span class="n">sym_dict</span><span class="p">(),</span><span class="n">key</span><span class="p">(),</span><span class="n">value</span><span class="p">()},</span>  
         <span class="k">begin</span>  
             <span class="k">try</span> <span class="nn">ec_dictionary</span><span class="p">:</span><span class="nb">get</span><span class="p">(</span><span class="nv">K</span><span class="p">,</span><span class="nn">ec_dictionary</span><span class="p">:</span><span class="n">add</span><span class="p">(</span><span class="nv">K</span><span class="p">,</span><span class="nv">V</span><span class="p">,</span><span class="nv">Dict</span><span class="p">))</span> <span class="k">of</span>  
                    <span class="nv">V</span> <span class="o">-&gt;</span>  
                        <span class="n">true</span><span class="p">;</span>  
                    <span class="p">_</span> <span class="o">-&gt;</span>  
                        <span class="n">false</span>  
             <span class="k">catch</span>  
                   <span class="p">_:_</span> <span class="o">-&gt;</span>  
                       <span class="n">false</span>  
             <span class="k">end</span>  
          <span class="k">end</span><span class="p">).</span>  
</pre></div>  
  
This property reads that for all dictionaries `get/2` using a key  
from a key/value pair just inserted using the `add/3` function  
will return that value. If that is not the case the property will  
evaluate to false.  
  
Running the property is done using `proper:quickcheck/1`:  
  
<div class="codehilite"><pre>proper:quickcheck<span class="o">(</span>ec_dictionary_proper:prop_get_after_add_returns_correct_value<span class="o">())</span>.  
....................................................................................................  
OK: Passed 100 <span class="nb">test</span><span class="o">(</span>s<span class="o">)</span>.  
<span class="nb">true</span>  
</pre></div>  
  
This was as expected, but at this point we will take a little detour  
and introduce a mistake in the `ec_gb_trees` implementation and see  
how that works.

