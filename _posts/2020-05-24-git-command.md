---
layout: post
title: "Display Github PR Number of Your Current Branch"
categories: [All, Technical]
tags: [git, github, command line]
fullview: false
excerpt: Show the PR number of your current branch
comments: true
---

To display the PR number of your current branch, you can run:

```bash
$ hub pr list -f "%I%n" -h "$(git rev-parse --abbrev-ref HEAD)"
12345
```

It's handy if you are working on multiple feature branches. Also, if your Heroku apps are named with Github PR numbers, this command can help to programmatically figure out the app's name.

This command is based on [hub](https://hub.github.com/) extension. For Mac users, you can install with 

```bash
$ brew install hub
```