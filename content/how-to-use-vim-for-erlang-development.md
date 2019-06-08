+++
author = "Tristan Sloughter"
type="post"
date = 2013-09-09T18:00:21Z
description = ""
draft = false
slug = "how-to-use-vim-for-erlang-development"
title = "How to use Vim for Erlang Development"

+++

[![vim editor logo](http://erlware.files.wordpress.com/2013/09/vim-logo-bas.png?w=150)](http://erlware.files.wordpress.com/2013/09/vim-logo-bas.png)  
  
<em>This post sponsored by [ErlangCamp 2013](http://www.erlangcamp.org) in Nashville which was epic!   
  
You are about to learn to use **Vim** as your editor for Erlang development. You will learn how to install and use a variety of really powerful Vim plugins to make Erlang dev with Vim smooth and satisfying!  
  
I have been developing Erlang now for about 13 years, many of them full time and even wrote a book on Erlang: [Erlang &amp; OTP in Action](http://manning.com/logan). I have loved every minute of it but there was always one thing that made me sad, probably makes you sad too - Emacs. Emacs is the de-facto editor for Erlang. The emacs mode included with the Erlang distro is quite wonderful. The fact still remains, Emacs, we do not like it. ctrl ~, ctrl x ctrl f etc... Nope!   
  
## Setting up Vim for Erlang  
  
Let's get started setting up Vim for Erlang development. The first thing we need to do is setup pathogen so that installing subsequent packages is really simple. The first thing to do is create the directory ` $HOME/.vim/autoload`. Download [pathogen.vim from here](https://github.com/tpope/vim-pathogen) and place it into this directory. Now add the following 2 commands to your `$HOME/.vimrc` file.    
  
`  
call pathogen#infect()  
call pathogen#helptags()  
`  
  
At this point pathogen will install and generate help documentation for any plugin you place into the `$HOME/.vim/bundle` directory - which you should of course create.   
  
With this created now we are ready to start installing plugins to make your life easier. Try these on for size by cloning these git repos directly into the `$HOME/.vim/bundle` directory. They will simply work next time you start vim.   
  
<table>  
<tr><td>[**vimerl.vim**](https://github.com/jimenezrick/vimerl)</td><td>Indenting, autocomplete and more for Erlang</td></tr>  
<tr><td>[**ctrlp.vim**](https://github.com/kien/ctrlp.vim)</td><td>ctrl p and open a powerful fuzzy file finder. Makes navigating file trees a thing of the past.</td></tr>  
<tr><td>[**NERDTree**](https://github.com/scrooloose/nerdtree)</td><td>Powerful file tree navigator right in vim - don't use it much since I installed ctrlp though.</td></tr>  
<tr><td>[**NERDTree Tabs**](https://github.com/jistr/vim-nerdtree-tabs)</td><td>Add the NERDTree file finder to all tabs you have open in vim.</td></tr>  
</table>  
  
Before we get into basics on how to use all these plugins to create Erlang magic I want to show you two bonus tricks I really love. First, get a better color scheme. To do this create the directory `$HOME/.vim/colors` and find yourself a slick color scheme to drop into it. I recommend [vividchalk.vim by TPope](https://github.com/tpope/vim-vividchalk).   
  
> **Pro Tip**>   
> For dropbox or other file sync users keep all your vim installs in sync easily like so; take your .vim and your .vimrc and move them into your Dropbox directory. Then run:>   
>   
> `>   
> ln -s ~/Dropbox/.vim ~/.vim>   
> ln -s ~/Dropbox/.vimrc ~/.vimrc>   
> `>   
>   
> Now all your machines vim installs will run just the same. If you have compatibility problems on any one, well then just skip this for that machine.  
  
Ok, so now on to how to use these plugins for Erlang/Vim greatness.   
  
## How to Use our Vim Plugins for Erlang Dev  
  
I am going to use the [source for Erlware Commons](https://github.com/erlware/erlware_commons) as an example. So I clone it first and then change into the `erlware_commons` directory and run vim. Now lets say I know what file I want to update, specifically the "ec_date.erl" file. The first thing I do is type ` p` and then start typing ec_date.erl.   
  
<pre>                                                                                                                                                            
~                                                                                 
[No Name] [TYPE= unix] [0/1 (100%)]                                               
&gt; test/ec_dictionary_proper.erl  
&gt; src/ec_dictionary.erl  
&gt; src/ec_date.erl                                                                 
 prt  path  ={ files }=  &gt;&gt; ec_da  
</pre>  
  
You can see that as I start typing and get to "ec_da" ctrlp has already displayed a narrowed down list of files in the directory tree under where I have opened vim that match. The file on the bottom ec_date.erl is the one selected and so just pressing enter here will open it. If I wanted to select "test/ec_dictionary_proper.erl" then I could simply press the up arrow and select it or keep typing until it was the only selection.  
  
Now, what if I don't know what file I want to select? This is where NERDTree comes into play. Run `:NERDTree` and you will pop open the file browser. Like this:  
  
<pre>  
  Press ? for help             |  
                               |~                                                 
.. (up a dir)                  |~                                                 
&lt;lang-projects/erlware_commons/|~                                                 
▸ doc/                         |~                                                 
▸ priv/                        |~                                                 
▸ src/                         |~                                                 
▸ test/                        |~                                                 
  CONTRIBUTING.md              |~                                                 
  COPYING                      |~                                                 
  Makefile                     |~                                                 
  README.md                    |~                                                 
  rebar.config                 |~                                                 
  rebar.config.script          |~                                                 
~                              |~                                                 
~                              |~                                                 
~                              |~                                                                                                                                         
</pre>  
  
Here we can see the directory tree for Erlware Commons. Each of the directories can be easily selected and expanded. Individual files can be selected and opened. There are a variety of ways to open a file. Below are the most common:   
  
*   `&lt;enter&gt;` will open the file in the right pane  
*   `T` will open in a new tab within vim and keep focus in NERDTree  
*   `t` will open in a new tab and bring focus to the new tab  
  
IF you want to see the NERDTree browser in all your tabs use `:NERDTreeTabsToggle` to toggle it on and off. It will be the exact same NERDTree in the exact same state and cursor position on all tabs - nice! Once you are focused on the code in a given tab and you want to jump back to the left and into the NERDTree pane use `&lt;ctrl&gt; ww`  
  
Once you have a load of tabs open you need to switch between then and to do this you need only two commands:  
  
*   `gt` will goto the next tab the next tab  
*   `gT` will goto the previous tab  
  
> **Pro Tip**>   
> Map the tab commands and the NERDTreeTabsToggle command by adding the following to your vimrc.>   
>   
> `>   
> map &lt;C-t&gt; :tabn&lt;Enter&gt;>   
> map &lt;C-n&gt; :tabnew&lt;Enter&gt;>   
> map nt :NERDTreeTabsToggle&lt;Enter&gt;>   
> `  
  
Ok, now on to editing Erlang with vimerl.   
  
## Editing with vimerl  
  
This is not going to be an exhaustive list of vimerl editing commands but just a few of the goodies. The 20% you will use 80% of the time.   
  
### Auto-indenting  
  
vimerl will auto-indent for you as you type. But if you come across a line that you want to indent try typing `==`. Lets say you want to indent a block of code. Simple, mark the line that starts the block with `ma` then go to the end of the block and tell vimerl to indent to the mark as such: `='a`. Now if your whole file is a mess then try `gg` to go to the beginning of your file and then `=G` to indent all the way to the end. You can do this all in one step as in `gg=G`.  
  
### Code Completion  
  
`ctrl-x ctrl-o` after typing a module name and a `:` will cause vimerl to suggest function names for you. It does this by searching the .beam and .erl files in the erlang code path (`code:get_path()` to see what they are) as well as looking at your rebar deps_dir if you are using rebar.config as part of your project.  
  
### Skeletons  
  
This is the feature that I loved most about the emacs mode for Erlang, well this and the auto indenting (most of the time, the fun() indenting still feels like a kick in the teeth). Here is a list of the most useful skeletons and the commands to generate them from within vimerl.  
  
*   **:ErlangApplication** generate the skeleton for an OTP application behaviour.  
*   **:ErlangSupervisor** generate the skeleton for an OTP supervisor behaviour.  
*   **:ErlangGen[Server|Fsm|Event]** skeletons for gen server, fsm and event - yay!  
  
Brilliant isn't it. Before I let you go there is one more invaluable command you should know about which is `:help vimerl` which will give you a list of all the other useful commands you may want to use. Remember to get it working be sure to add `call pathogen#helptags()` to the top of your .vimrc file. Goodbye Emacs, welcome back old friend Vim.   
  
Follow me on twitter [@martinjlogan](http://twitter.com/martinjlogan)  
  
&lt;esc&gt;:wq

