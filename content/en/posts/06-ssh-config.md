+++
title = "SSH Config: The file you've been ignoring"
author = "Felipe A."
draft = false
tags = [
    "Felipe A.",
]
date = "2025-03-17"
categories = [
    "DevTools",
]
menu = "main"
+++

### Are you still typing full SSH commands like it's 2005?

![meme](/images/typing-furiously.gif)

We've all been there—typing out the same SSH commands over and over:

```bash
ssh -i ~/.ssh/special_key.pem user@ec2-54-234-56-78.compute-1.amazonaws.com
```

### Enter the magical ~/.ssh/config file

```
# ~/.ssh/config
Host bastion
    HostName ec2-54-234-56-78.compute-1.amazonaws.com
    User ec2-user
    IdentityFile ~/.ssh/special_key.pem

Host prod-db
    HostName 10.0.5.12
    User admin
    ProxyJump bastion
```

Now you can simply type:

```bash
ssh bastion
# or
ssh prod-db
```

### Why this is life-changing:

- **Human-readable aliases** instead of IP addresses
- **Automatic key selection** without the -i flag
- **Jump hosts** configured once, used forever

Your future self will thank you for taking 5 minutes to set this up today!
