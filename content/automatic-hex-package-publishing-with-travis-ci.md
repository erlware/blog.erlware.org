+++
author = "Tristan Sloughter"
date = 2015-11-19T05:31:57Z
description = ""
draft = false
slug = "automatic-hex-package-publishing-with-travis-ci"
title = "Automatic Hex Package Publishing with Travis-CI"

+++

**Note**: [Travis CI](https://travis-ci.org) does not yet provide [rebar3](https://rebar3.org) as a build tool for Erlang projects, so you will have to include a rebar3 executable escript in your repository.

In your application's `.app.src` file use `"git"` as the version. This will replace the version string with the current tag when publishing the hex package:

```erlang
{vsn,"git"},
```

In the `.travis.yml` configuration add the below section so that the script `scripts/hex.sh` is run everytime a new tag is made:

```
deploy:
  provider: script
  script: scripts/hex.sh
  on:
    tags: true
```

Create the script `hex.sh` under `scripts/` and make it executable:

```bash
#!/usr/bin/env bash

## Setup hex user
mkdir -p ~/.hex
echo '{username,<<"'${HEX_USERNAME}'">>}.' > ~/.hex/hex.config
echo '{key,<<"'${HEX_KEY}'">>}.' >> ~/.hex/hex.config

## Add the rebar3 hex plugin to global
mkdir -p ~/.config/rebar3
echo '{plugins, [rebar3_hex]}.' > ~/.config/rebar3/rebar.config

./rebar3 hex publish <<EOF
y
EOF
```

In the script it relies on two environmental variables `$HEX_USERNAME` and `$HEX_KEY`. Using either the Travis CLI tool or web interface you can create these variables and have the values encrypted or unseen by viewers of your repository. For instructions see the Travis-CI [environment variables docs](http://docs.travis-ci.com/user/environment-variables/).

