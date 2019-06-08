+++
author = "Tristan Sloughter"
type="post"
date = 2012-05-09T20:01:52Z
description = ""
draft = false
slug = "erlang-common-test-continuous-integration"
title = "Erlang Common Test Continuous Integration"

+++

[Common Test](http://www.erlang.org/doc/apps/common_test/index.html) is a well thought out integration testing framework for Erlang. If you   
are not using it you probably should be. However, it has one fault. It  
does not return non-negative exit status' to the caller when the tests  
fail. This is a major oversight, and it makes it difficult to use as  
part of a continuous integration scheme or in a make based build  
system.  
  
The long term fix is for the OTP folks to resolve the issue in the  
`ct_run` command. To that end I have filed a bug report with the  
Erlang folks. In the short term, though, we need this behaving  
correctly. After much twiddling around with different solutions and  
conversions on the erlang-questions list. This solution finally popped  
out of a conversation with Lukas Larsson. Basically, we use the old  
unix standby of awk.  
  
<pre>  
  
    ct_run -dir tests  ... | awk "/FAILED/{exit 1;}/failed/{exit 1;}/SKIPPED/{exit 1;}"  
  
</pre>  
Where `...` is replaced with your additional options. Its not the best  
solution on the planet, but it is the simplest one that I found that  
works consistently.

