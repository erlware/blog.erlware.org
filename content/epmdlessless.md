+++
author = "Tristan Sloughter"
type="post"
categories = ["Erlang"]
date = 2020-12-02T10:41:00Z
description = ""
draft = false
slug = "epmdlessless"
tags = ["Erlang"]
title = "Running Erlang Releases without EPMD on OTP 23.1+"
+++

Erlang/OTP deployments that want to provide shell access or cluster nodes relied
on something called the Erlang Port Mapper Daemon (EPMD), a separate process
that handled all distribution mechanisms. Previous attempts at working around
this requirement used third party dependencies such as
[epmdless](https://github.com/tsloughter/epmdless) and required specific
configuration arguments to the VM to setup. This was not ideal and was easy to
get wrong.

With Erlang/OTP 23.1 and relx 4.2 (included in Rebar3 3.14.3) it is now possible
to conveniently run Erlang releases without relying on EPMD for opening a
remote shell or clustering nodes. This is particularly going to make Erlang releases
simpler to configure in locked down environments, and easier to use with containers.

EPMD is a separate process that is started, if
not already running, when an Erlang node boots with distribution enabled --
meaning `-sname` or `-name` is passed to the boot command. The new node [talks to
the EPMD process](http://erlang.org/doc/apps/erts/erl_dist_protocol.html#epmd-protocol) to register its name with a port that the node will also bind
to. When a node wants to connect to another one it must look up the port the other
node is listening on through EPMD. This is required both for
clustering nodes together and connecting a remote shell on the command line --
which is essentially the same as clustering because a new node is started before
attaching to the remote node to bring up a shell on that node.

Bypassing EPMD has been made possible through a couple new options added to `erl`:

* [`-erl_epmd_port P`](https://github.com/erlang/otp/commit/c21bbb6136a1f4d343c3cf53476107e78221a68f): Configures the port this node will listen on and use when
  connecting to remote nodes.
* [`-dist_listen false`](https://github.com/erlang/otp/commit/7a7c90be0e87cb3b4920de5aaf215c4b9cebcb30): Setting this keeps the node from binding to the port,
  instead it is only used for connecting to remote nodes. The use case for this
  is when opening a remote shell `-remsh <node>` (`remote_console` in the
  release script) to a local node.
* [`-[s]name
  undefined`](https://github.com/erlang/otp/commit/61b1ad3c57f4e92fb9b55f97b9ffd9dee80067e2):
  Passing `undefined` for the short or long name of a node generates a unique node
  name and sets additional flags, `-dist_listen false -hidden -dist_auto_connect never`.

An extra option has also been added to [`erl_call`](http://erlang.org/doc/man/erl_call.html) :

* [`-address
  P`](https://github.com/erlang/otp/commit/ce4fcf0640d81a268c15af339a888406b757ced5):
  Sets the address and/or port to connect to the running Erlang node. This
  removes the need to look up the remote node or to even specify its name when
  running `erl_call`.
* [`-R`](https://github.com/erlang/otp/commit/3a57ed212befae5d0e03569408849e8a72122911): Tells `erl_call` to use a dynamic random name for the hidden node it starts.
  
As of OTP 23.1+ the `relx` generated release script uses `erl_call` instead of
the custom `escript` `nodetool` when the commands `rpc` and `eval` are used to
run code on a running node.

These options are used automatically by the release shell script when starting a
node, opening a remote shell or interacting with the running node through `rpc`
and `eval`. The only change you as a user must make to run your release without
`epmd` is to set the OS environment variable `ERL_DIST_PORT`.

When `ERL_DIST_PORT` is set in the environment the option to not start EPMD
(`-start_epmd false`) is added to the `erl` arguments automatically, along with
`-erl_epmd_port ${ERL_DIST_PORT}`. The `ERL_DIST_PORT` is also used by
`erl_call` with `-address ${ERL_DIST_PORT}`.

[epmdlessless](https://github.com/tsloughter/epmdlessless) is an example project
that comes with a
[docker-compose.yml](https://github.com/tsloughter/epmdlessless/blob/main/docker-compose.yml)
setup that makes it simple to play these new features. Note that no changes are
required to `vm.args.src` to disable EPMD or set a port to use, the only
configuration done is through the environment variable `ERL_DIST_PORT` in
`docker-compose.yml`:

```
    ...
    environment:
      - ERL_DIST_PORT=8001
    ...
```

Note that each node must listen on the same port because setting
`-erl_epmd_port P` tells Erlang both what port to listen on and what port to use
when connecting to other nodes.

After cloning the repo, build the Docker image and start 3 nodes (`node_a`, `node_b`, `node_c`):

```
$ docker-compose up
```

Then, use `docker exec` to open a remote shell on `node_a` and connect to `node_b`:

```
$ docker exec -ti node_a bin/epmdlessless remote_console
Erlang/OTP 23 [erts-11.1.3] [source] [64-bit] [smp:8:8] [ds:8:8:10] [async-threads:1] [hipe]

Eshell V11.1.3  (abort with ^G)
(epmdlessless@node_a)1> net_kernel:connect_node(epmdlessless@node_b).
true
(epmdlessless@node_a)2> erlang:nodes().
[epmdlessless@node_b]
```

Do the same on `node_c` and also connect to `node_b` and you'll see the full
mesh is created with `node_a`:

```
$ docker exec -ti node_c bin/epmdlessless remote_console
Erlang/OTP 23 [erts-11.1.3] [source] [64-bit] [smp:8:8] [ds:8:8:10] [async-threads:1] [hipe]

Eshell V11.1.3  (abort with ^G)
(epmdlessless@node_c)1> net_kernel:connect_node(epmdlessless@node_b).
true
(epmdlessless@node_c)2> erlang:nodes().
[epmdlessless@node_b,epmdlessless@node_a]
```

Test the `rpc` command by running `erlang:nodes()` on `node_b`:

```
$ docker exec -ti node_b bin/epmdlessless rpc erlang nodes
[epmdlessless@node_a, epmdlessless@node_c]
```

Lastly, you can verify that none of the commands caused EPMD to be run:

```
$ docker exec -ti node_b erts-11.1.3/bin/epmd -names
epmd: Cannot connect to local epmd
```

## Separate Ports: Where epmdless is Still Needed

It is important to note that this requires every node to listen on the same
port, which requires every node to have its own IP. If this isn't an option, or
for whatever reason you need to use separate ports for nodes, then [epmdless](https://github.com/tsloughter/epmdless)
is an option to replace EPMD and track the node to port mapping. But if you are
running with containers in an environment like Kubernetes you can now keep EPMD
from running in each container by simply upgrading Erlang and Rebar3, and
setting the `ERL_DIST_PORT` environment variable.
