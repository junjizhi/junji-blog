---
layout: post
title: "Back to Emacs"
categories: [All, Experience]
tags: [emacs, doom emacs]
fullview: false
excerpt: My experience, learnings and ❤️ with Emacs
comments: true
---

## My History with Emacs

-   Started using vanilla Emacs in 2012
-   Copied a bunch of configs from the Internet
-   Switched to Spacemacs in 2017
-   Tried Org mode, but notes search or Babel never worked well for me
-   Switched to VSCode in 2020 for a year
-   Tried Doom Emacs in July 2021, fell in love and switched back since then
-   Now I do my coding editing, and notes writing in Emacs every day


## Learnings

-   Bare metal Emacs is too hard to use and forces me to tinker all the time. After my config bankruptcy, [Doom Emacs](https://github.com/hlissner/doom-emacs) saves my life.
-   I bound a lot of two-key combo with the Mac OS keyboard&rsquo;s `Command/Super` key, like `Super-t` -> `(counsel-buffer-or-recentf)`. But later I found it hard to switch to Linux desktops because the keyboard layout is different from Apple. So I started remembering function names instead, and relied on Ivy autocomplete to help me. That reduces my reliance on Apple hardware.
-   Org mode + Babel, when it WORKS, it is powerful! Doom Emacs shines with org-mode working out of box.
-   For packages backed by Linux commands, like `Magit` or `counsel-ag`, it is important to learn the underlying model (`git`) or CLI options (`ag`) first. Otherwise, your usage is quite limited and never realizes their potential.


<a id="org435cd3c"></a>

## Personal workflow

-   I like to export an org file into read-only html file a lot. I find it reading my notes in a different fonts and background is refreshing and inspiring. So I used `C-c C-e h o` a lot. If the default HTML template is too plain to you. Grab one theme from [org-html-themes](https://github.com/fniessen/org-html-themes) and roll with it.
-   I use a `wm.org` like a scratch pad and a brain extension. This is inspired by [Cal Newport&rsquo;s WorkingMemory.txt](https://www.calnewport.com/blog/2015/10/27/deep-habits-workingmemory-txt-the-most-important-productivity-tool-youve-never-heard-of/). **Our lives have so many important decisions to make, but worring where to take notes is NOT one of them**.
- When editing org files, I like to narrow to the subtree, so my screen is distraction-free, whic helps my focus a lot.

<a id="org8e09376"></a>

## My favorite packages

-   Doom-emacs (More of a framework than a package)
-   Magit
-   Org-mode
-   counsel-projectile (as opposed to Helm which I found too heavy)


