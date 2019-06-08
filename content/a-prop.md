+++
author = "Tristan Sloughter"
type="post"
categories = ["Erlang", "proper"]
date = 2019-06-08T09:08:10Z
description = ""
draft = false
slug = "a-prop"
tags = ["Erlang", "proper", "property testing"]
title = "A Little on Property-Based Testing with PropEr"
+++

Fred Hebert's latest book [Property-Based Testing with PropEr, Erlang and Elixir](http://propertesting.com/) is out in a print version from The Pragmatic Programmers.

If you are like me you've known you need to figure out property testing for a long time now and keep putting it off. Now that there is a book, with a free version even, it is the best time to get going.

The book even details avoiding a common pitfall that I certainly fall into anytime I've tried picking up  property based testing: attempting to shoehorn property testing in to any problem even when a regular unit test is a better fit.

As is made clear in the book, practice is important to leveling up with property based testing. So it is great that the book includes many exercises based on useful projects – not just sorting a list like the classic property test we've all seen and not been able to get beyond!

One thing I have to add to the content is running the [properties through Common Test](https://erldocs.com/current/common_test/ct_property_test.html). You still want to be able to run them on their own with the PropEr plugin so you can fiddle with the options easily but I find it far better to also include the tests in a CT suite to keep all my tests in a single command and output in CI, with a single cover configuration and nice rendered junit based output in services like CircleCI.

An example of this method is found in this [CT Suite](https://github.com/census-instrumentation/opencensus-erlang/blob/e45bd01e27f666fbf17453a39594376a1f1d601e/test/oc_sampler_period_or_count_prop_SUITE.erl) for `opencensus-erlang`. But it basically looks like:

```
init_per_suite(Config) ->
    [{property_test_tool, proper} | Config].

prop_test(Config) ->
    ct_property_test:quickcheck(prop_base:prop_test(), Config).
```

The only issue is that `ct_property_test` wants to compile the tests and rebar3 already takes care of that. To resolve this I have a PR, so if you agree please give a thumbs up to my PR [https://github.com/erlang/otp/pull/2145](https://github.com/erlang/otp/pull/2145) if it hasn't been merged yet when you are reading this :)

So do yourself a favor and at least checkout [http://propertesting.com/toc.html](http://propertesting.com/toc.html), or even better buy the [book](https://pragprog.com/book/fhproper/property-based-testing-with-proper-erlang-and-elixir).

