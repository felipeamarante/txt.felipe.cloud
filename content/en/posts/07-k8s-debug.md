+++
title = "Kubernetes: Debug Pods Like a Pro"
author = "Felipe A."
draft = false
tags = [
    "Felipe A.",
]
date = "2025-02-18"
categories = [
    "K8s",
]
menu = "main"
+++

### Ever stared at a CrashLoopBackOff wondering what went wrong?

![meme](/images/evergreen.gif)

We've all been there: looking at a failing pod with no idea why it's not starting. The logs are empty, and you're about to lose your mind.

### Enter the ephemeral debug container

Kubernetes 1.23+ has a feature worth knowing about:

```bash
kubectl debug -it failing-pod-xyz --image=busybox --target=failing-container
```

This attaches a debug container to your existing pod without restarting it. You can inspect the filesystem, check environment variables, and see what's actually happening inside.

### But what if the pod won't even start?

Try this one-liner to create a clone with the same volumes but a different image:

```bash
kubectl debug failing-pod-xyz -it --copy-to=debug-pod --container=debug-container --image=ubuntu
```

### A reusable debug pod

Create this YAML and keep it handy:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: debug-toolkit
spec:
  containers:
  - name: debug
    image: nicolaka/netshoot
    command: ['sleep', '999999']
  tolerations:
  - operator: "Exists"
```

Apply it with:

```bash
pbpaste | kubectl apply -f-
```

You get a pod with most networking and debugging tools you'd actually need, and it can run on any node.

### kubectl-debug plugin

```bash
kubectl krew install debug
kubectl debug pod failing-pod-xyz
```

You can finally see what's happening inside your pods instead of guessing.
