+++
author = "Tristan Sloughter"
type="post"
categories = ["Erlang", "rebar3", "relx", "OTP"]
date = 2018-06-21T03:55:53Z
description = ""
draft = false
slug = "otp-21-new-sys_config_src-option-in-relx"
tags = ["Erlang", "rebar3", "relx", "OTP"]
title = "OTP-21: New sys_config_src option in relx"

+++

With the release of OTP-21 and rebar3 3.6.0, which includes a new relx 3.25.0, there is better support for dynamic configuration of releases at runtime. If you are new to this concept checkout the [rebar3 docs first](http://www.rebar3.org/docs/releases#section-dynamic-configuration)

No longer will you be forced to declare `RELX_REPLACE_OS_VARS` and convert strings to integers in your code because the `sys.config` file had to be valid Erlang terms when building the release.

Instead, you can use `{sys_config_src, "config/sys.config.src"}` and `{vm_args_src, "config/vm.args.src"}` and the extended start up script will replace variables for `.src` files without the need to set `RELX_REPLACE_OS_VARS`.

Since this is based on a change in `systools` to support including `sys.config.src` without requiring validation of the contents you can also freely declare a config file that looks like this:

```
[{myapp, [{port, ${PORT}}]}].
```

An error, because this isn't a valid Erlang term file, will still be returned if your release configuration uses the `sys_config` option, but with `sys_config_src` it does not have to be valid terms until after variable replacement. In this case, `PORT=8080 bin/myrel console` will use a `sys.config` with contents `[{myapp, [{port, 8080}]}].`

`vm_args_src` was added to relx for consistency. The only thing that changes is how the start script handles it internally and that `RELX_REPLACE_OS_VARS` is not required for replacement to be run.


This change also will resolve at least a couple annoyances related to how relx had to handle `sys.config` before, https://github.com/erlang/rebar3/issues/1753 and https://github.com/erlware/relx/issues/604

