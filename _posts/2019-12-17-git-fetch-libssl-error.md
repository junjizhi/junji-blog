---
layout: post
title: "dyld: Library not loaded: /usr/local/opt/openssl/lib/libcrypto.1.0.0.dylib"
categories: [All, Technical]
tags: [git, openssl]
fullview: false
excerpt: My solution for the link missing issue. The popular solution in Github isn't correct.
comments: true
---

*UPDATE: If you are having issue with `libssl.1.0.0.dylib` instead of `libcrypto.1.0.0.dylib` (mind the difference), please try the solution in [Stackoverflow](https://stackoverflow.com/a/59184347/8175889): `brew switch openssl 1.0.2t`.*

Recently, Homebrew drops OpenSSL v1.0 for v1.1. The popular [TagUI Github
issue](https://github.com/kelaberetiv/TagUI/issues/86) suggests downgrading
openssl to v1.0 because their stack hasn't support v1.1 yet. But that's not a
good idea in general. This posts presents my solution to the specific issue I saw.

When I reinstalled openssl v1.1, git suddently not working:

```shell
$ git fetch origin
dyld: Library not loaded: /usr/local/opt/openssl/lib/libcrypto.1.0.0.dylib
  Referenced from: /usr/local/bin/ssh
  Reason: image not found
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

When you read the message, it was quite clear: `ssh` is not working. My solution is as simple as

```shell
brew upgrade openssh
```

The new openssh version works fine with openssl v1.1
```shell
$ brew info openssh                                                                                                                                             [10:29:34]
openssh: stable 8.1p1 (bottled)
OpenBSD freely-licensed SSH connectivity tools
https://www.openssh.com/
/usr/local/Cellar/openssh/8.1p1 (45 files, 4.7MB) *
  Poured from bottle on 2019-12-18 at 10:28:20
From: https://github.com/Homebrew/homebrew-core/blob/master/Formula/openssh.rb
==> Dependencies
Build: pkg-config ✔
Required: ldns ✔, openssl@1.1 ✔
==> Analytics
install: 4,936 (30 days), 17,823 (90 days), 60,969 (365 days)
install-on-request: 4,725 (30 days), 16,718 (90 days), 57,024 (365 days)
build-error: 0 (30 days)
```

