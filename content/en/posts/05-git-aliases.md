+++
title = "The git alias that changed my life"
author = "Felipe A."
draft = false
tags = [
    "Felipe A.",
]
date = "2025-03-14"
categories = [
    "DevTools",
]
menu = "main"
+++

### Ever get lost in your git log?

![meme](/images/lost-in-code.png)

We've all been there—trying to figure out what happened in a project by staring at the default `git log` output. It's like trying to read a novel written on receipt paper.

After years of squinting at commit hashes and timestamps, I finally found the solution that changed everything.

### The one git alias you actually need

```
git config --global alias.lg 'log --graph --pretty=format:"%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset" --abbrev-commit'
```

Now just type `git lg` and behold:

![git-log](/images/git-log.png)

### What makes this so good?

- **Visual branching** with the `--graph` option
- **Color-coded** information that's actually readable
- **Compact format** showing just what you need
- **Branch references** clearly visible
- **Relative time** so you know if it was "2 days ago" instead of some timestamp

### How to set it up?

Just copy and paste that one command into your terminal. That's it.

If you prefer, add it directly to your `.gitconfig` file:

```
[alias]
    lg = log --graph --pretty=format:"%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset" --abbrev-commit
```

Now you'll never get lost in your git history again. You're welcome.