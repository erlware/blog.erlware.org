+++
author = "Tristan Sloughter"
categories = ["Emacs", "Erlang"]
date = 2012-05-16T01:39:32Z
description = ""
draft = false
slug = "getting-flymake-and-rebar-to-play-nice"
tags = ["Emacs", "Erlang"]
title = "Getting Flymake and Rebar to Play Nice"

+++

**TLDR;**  
Copy and paste the following into your elisp erlang-mode configuration to get flymake working with Rebar projects.  
  
<pre>  
 (defun ebm-find-rebar-top-recr (dirname)  
      (let* ((project-dir (locate-dominating-file dirname "rebar.config")))  
        (if project-dir  
            (let* ((parent-dir (file-name-directory (directory-file-name project-dir)))  
                   (top-project-dir (if (and parent-dir (not (string= parent-dir "/")))  
                                       (ebm-find-rebar-top-recr parent-dir)  
                                      nil)))  
              (if top-project-dir  
                  top-project-dir  
                project-dir))  
              project-dir)))  
  
    (defun ebm-find-rebar-top ()  
      (interactive)  
      (let* ((dirname (file-name-directory (buffer-file-name)))  
             (project-dir (ebm-find-rebar-top-recr dirname)))  
        (if project-dir  
            project-dir  
          (erlang-flymake-get-app-dir))))  
  
     (defun ebm-directory-dirs (dir name)  
        "Find all directories in DIR."  
        (unless (file-directory-p dir)  
          (error "Not a directory `%s'" dir))  
        (let ((dir (directory-file-name dir))  
              (dirs '())  
              (files (directory-files dir nil nil t)))  
            (dolist (file files)  
              (unless (member file '("." ".."))  
                (let ((absolute-path (expand-file-name (concat dir "/" file))))  
                  (when (file-directory-p absolute-path)  
                    (if (string= file name)  
                        (setq dirs (append (cons absolute-path  
                                                 (ebm-directory-dirs absolute-path name))  
                                           dirs))  
                        (setq dirs (append  
                                    (ebm-directory-dirs absolute-path name)  
                                    dirs)))))))  
              dirs))  
  
    (defun ebm-get-deps-code-path-dirs ()  
        (ebm-directory-dirs (ebm-find-rebar-top) "ebin"))  
  
    (defun ebm-get-deps-include-dirs ()  
       (ebm-directory-dirs (ebm-find-rebar-top) "include"))  
  
    (fset 'erlang-flymake-get-code-path-dirs 'ebm-get-deps-code-path-dirs)  
    (fset 'erlang-flymake-get-include-dirs-function 'ebm-get-deps-include-dirs)  
</pre>  
  
**Intro**  
  
Its probably no great surprise to anyone that I dislike [Rebar](https://github.com/basho/rebar "Rebar") a lot. That said there are times when I have no choice but to use it. This is always either because a company I am contracting for uses it, or an open source project I am contributing to uses it. When I am forced to use it there are a few things I don't want to give up. Most important among these is [Flymake](http://flymake.sourceforge.net/ "Flymake") for Erlang. The default setup for Flymake doesn't work for Rebar projects because Flymake does not know where the code and include paths for dependencies are. Fortunately, we can fix this with a few lines of elisp.  
  
**Flymake For Erlang**  
  
First make sure you have Flymake for Erlang installed. It is easiest just [to follow the directions available on the Erlang Website](http://www.erlang.org/doc/apps/tools/erlang_mode_chapter.html "Erlang Flymake Instructions").  
  
**The Elisp Additions for Erlang Flymake**  
  
There are two defvars that point to functions that are used to search for the correct code paths and include paths respectively. We are going to replace those functions with our own functions. Both these functions search upwards from the directory that contains the file pointed to by the current buffer, looking for the top most 'rebar.config' in the directory path. It then uses that for a base and searches down the directory structure looking for either 'ebin' files or 'include' files.  
  
There are two things to note here. The first is that you must have already run `get-deps` for rebar for this to work and the second is that if your project is truly huge or you have way more dependencies then you probably need this search could take a second or two. That is a second or two too long in an interactive compiler like Flymake. That said, the likelihood that you will run into this second problem is quite low.  
  
**Getting Started**  
  
The very thing you want to do is ensure that you have `required` the `erlang-flymake` module. Most of what we do below depends on this.  
  
<pre>  
(require 'erlang-flymake)  
</pre>  
  
**Finding the Top rebar.config**  
  
The second thing we want to do is look for the top `rebar.config` in the project. If a rebar project contains more then one OTP application its quite likely that it will contain more then one `rebar.config`. The very topmost `rebar`config` is the right one to serve as root of our search. So we introduce a set of recursive functions to look for that top level dir.  
  
<pre>  
    (defun ebm-find-rebar-top-recr (dirname)  
      (let* ((project-dir (locate-dominating-file dirname "rebar.config")))  
        (if project-dir  
            (let* ((parent-dir (file-name-directory (directory-file-name project-dir)))  
                   (top-project-dir (if (and parent-dir (not (string= parent-dir "/")))  
                                       (ebm-find-rebar-top-recr parent-dir)  
                                      nil)))  
              (if top-project-dir  
                  top-project-dir  
                project-dir))  
              project-dir)))  
</pre>  
  
`ebm-find-rebar-top-recr` will return either the top most directory or `nil`. Our next function takes that result and does something useful with.  
  
<pre>  
    (defun ebm-find-rebar-top ()  
      (interactive)  
      (let* ((dirname (file-name-directory (buffer-file-name)))  
             (project-dir (ebm-find-rebar-top-recr dirname)))  
        (if project-dir  
            project-dir  
          (erlang-flymake-get-app-dir))))  
</pre>  
  
In this function, we get the directory containing the file pointed at by the current buffer. We then call our recr function. If it returns a directory we return that, if it returns nil however, we call the original `erlang-flymake-get-app-dir` function.  
  
At this point we should have our project root. Now its a simple matter of recursively searching down the directory tree looking for files of a certain name. So we create a function that does just that, given a directory and a name will return a list of absolute paths for each subdirectory that matches the specified name.  
  
<pre>  
(defun ebm-directory-dirs (dir name)  
        "Find all directories in DIR."  
        (unless (file-directory-p dir)  
          (error "Not a directory `%s'" dir))  
        (let ((dir (directory-file-name dir))  
              (dirs '())  
              (files (directory-files dir nil nil t)))  
            (dolist (file files)  
              (unless (member file '("." ".."))  
                (let ((absolute-path (expand-file-name (concat dir "/" file))))  
                  (when (file-directory-p absolute-path)  
                    (if (string= file name)  
                        (setq dirs (append (cons absolute-path  
                                                 (ebm-directory-dirs absolute-path name))  
                                           dirs))  
                        (setq dirs (append  
                                    (ebm-directory-dirs absolute-path name)  
                                    dirs)))))))  
              dirs))  
</pre>  
  
Now we write a couple of functions to replace the corresponding functions in `erlang-flymake`. The first looks for all `ebin` directories while the second looks for all `include` directories.  
  
<pre>  
    (defun ebm-get-deps-code-path-dirs ()  
        (ebm-directory-dirs (ebm-find-rebar-top) "ebin"))  
  
    (defun ebm-get-deps-include-dirs ()  
       (ebm-directory-dirs (ebm-find-rebar-top) "include"))  
</pre>  
  
Finally we replace the `erlang-flymake` versions of those functions with our implementations.  
  
<pre>  
(fset 'erlang-flymake-get-code-path-dirs 'ebm-get-deps-code-path-dirs)  
(fset 'erlang-flymake-get-include-dirs-function 'ebm-get-deps-include-dirs)  
</pre>  
  
**Conclusion**  
  
This approach is a bit of a hack, we basically use some heuristics to find a root and then just grab everything under that that looks remotely like a code or include directory. While its a bit hacky it has the valuable upside that its flexible and robust.

