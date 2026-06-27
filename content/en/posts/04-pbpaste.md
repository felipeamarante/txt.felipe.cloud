+++
title = "How to ctrl+v straight to your k8s cluster on mac"
author = "Felipe A."
draft = false
tags = [
    "Felipe A.",
]
date = "2025-01-25"
categories = [
    "K8s",
]
menu = "main"
+++

### So you are following a long tutorial...

And you are getting bloody tired of copying and pasting all the YAMLs... urgh.  
What if I told you I can make your work 50% less painful? Would you pay me a coffee? ☕

Here's the YAML:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: definitely-not-prod
  labels:
    owner: "friday-5pm-dev"
spec:
  containers:
    - name: chaos-container
      image: busybox
      command: ["sh", "-c", "echo 'Deploying... just kidding!' && sleep 3600"]
      lifecycle:
        postStart:
          exec:
            command: ["/bin/sh", "-c", "echo 'Context set correctly?'"]
        preStop:
          exec:
            command: ["/bin/sh", "-c", "echo 'Goodbye, cruel cluster!'"]
  restartPolicy: Never
  nodeSelector:
    coffee-supply: "critical"
```

Copy it, then pipe your clipboard straight into kubectl:

```bash
pbpaste | k apply -f-
```

Tadá! Works like a charm

![pbpaste](/images/pbpaste.png)

Now be careful with what you copy 🫡
