+++
title = " Easy way to handle aws cli multi profile"
author = "Felipe A."
draft = false
tags = [
    "Felipe A.",
]
date = "2025-01-17"
categories = [
    "AWS",
]
menu = "main"
+++

### Have you ever run a command on the wrong account?

![meme](/images/this-is-fine.gif)

It happened once—I deployed a production stack into the sandbox account and had no idea why it wasn't working... Turning this frustrating experience into something useful, I decided to build a tool that helps me (and potentially you) set a profile or context, similar to [kubectx](https://github.com/ahmetb/kubectx).

It's a simple shell function that you add to your `.bashrc` or `.zshrc`, and it loads whenever you start a new shell session. It's called `awsctx` (yes, very creative).

![awsctx](/images/awsctx.png)

### How do I install it?

Please refer to my well-documented README.md in this repo! And don't forget to star it.

https://github.com/felipeamarante/awsctx
