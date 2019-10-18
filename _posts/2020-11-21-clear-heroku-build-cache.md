---
layout: post
title: "The Right Way to Clear Heroku Build Cache"
categories: [All, Technical]
tags: [heroku, heroku build cache, heroku build, heroku slug size]
fullview: false
excerpt: The ThoughtBot article suggests a command that doesn't work. The one from Heroku official answer works and solves my headache.
comments: true
---
If Heruku fails to build and complains about:

> Compiled slug size: 500.3M is too large (max is 500M)

you may Google and find the [ThoughtBot article](https://thoughtbot.com/blog/how-to-reduce-a-large-heroku-compiled-slug-size) which suggests:

```bash
# This command DOES NOT clear build cache
$ heroku repo:purge_cache --app your-app-name
```

Even after running the command, the build size didn't get reduced at all.

Instead, the [Heroku official answer](https://help.heroku.com/18PI5RSY/how-do-i-clear-the-build-cache) has the **CORRECT** command:

```bash
$ heroku plugins:install heroku-builds
$ heroku builds:cache:purge --app your-app-name
```

What confused me was the CLI documentation about `heroku repo:purge_cache` is

> This will delete the contents of the build cache stored in the repository.

while `heroku builds:cache:purge` doc says:

> purge the build cache for the specified app

But only the latter does its job well.