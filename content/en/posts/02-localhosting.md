+++
title = "How to expose your localhost"
author = "Felipe A."
draft = false
tags = [
    "Felipe A.",
]
date = "2025-03-05"
categories = [
    "K8s",
]
menu = "main"
+++





### Hey, take a look at my website..




![meme](/images/localhost01.png)



Recently I was deploying something to my kind kubernetes cluster and wondered how I could show the work to my colleague. In the past I played with [ngrok](https://ngrok.com) to expose my localhost to the interwebs, but on kubernetes I would need to do some portforwarding kung-fu that was not an option.

Then I saw that [Ngrok launched an operator](https://github.com/ngrok/ngrok-operator) that can easily expose your dodgy containers to the web with a simple creation of a Ingress.


![ngrok](/images/localhost02.png)

It's pretty simple to use, steps are,
 1. Create an [Ngrok free account](https://dashboard.ngrok.com/signup)
 1. [Install the operator](https://ngrok.com/docs/getting-started/kubernetes/?k8s-install=ingress#step-1-install-kubernetes-operator)
 1. Deploy whatever you want to expose - Example, [Podinfo](https://github.com/stefanprodan/podinfo)
 1. Create an ingress like the one below, just replace the host with your Ngrok **full Domain Name**


 ```yaml
 apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  ingressClassName: ngrok
  rules:
    - host: YOUR_NGROK_DOMAIN_HERE
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: example-service
                port:
                  number: 80
```

Wait a few seconds and you should have the URL when you describe the ingress you just created

```bash
kubectl get ing -A
```

Happy localhosting.
