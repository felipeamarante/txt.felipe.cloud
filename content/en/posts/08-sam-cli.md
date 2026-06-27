+++
title = "SAM CLI: Serverless Made Simple (or million lines terraform?)"
author = "Felipe A."
draft = false
tags = [
    "Felipe A.",
]
date = "2025-02-26"
categories = [
    "AWS",
]
menu = "main"
+++

### Still deploying Lambda functions by hand?

![meme](/images/bbq-sim.gif)

We've all been there: manually zipping code, uploading to S3, configuring triggers, setting up API Gateway... it's enough to make you question if serverless is actually worth it. Or worse, writing hundreds of lines of Terraform just to deploy a simple function.

### Enter AWS SAM CLI

AWS Serverless Application Model (SAM) CLI cuts through all of that. Think of it as Docker Compose for Lambda functions.

```bash
# Install it
brew install aws-sam-cli
```

### The 30-second serverless app

Create a new project:

```bash
sam init
```

Choose the "Hello World Example" with Python, and you'll get a project structure like this:

```
my-app/
├── hello_world/
│   ├── __init__.py
│   ├── app.py
│   └── requirements.txt
├── events/
│   └── event.json
├── template.yaml
└── tests/
```

`template.yaml` is where your entire application lives:

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Resources:
  HelloWorldFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: hello_world/
      Handler: app.lambda_handler
      Runtime: python3.9
      Events:
        HelloWorld:
          Type: Api
          Properties:
            Path: /hello
            Method: get
```

### Test locally (yes, really!)

```bash
sam local invoke
```

Or start a local API (Requires docker):

```bash
sam local start-api
```

### Deploy with a single command

```bash
sam deploy --guided
```

One command builds your code, packages it, creates or updates the CloudFormation stack, and hands you an API endpoint.

### Pro tip: Use SAM Accelerate for faster iterations

```bash
sam sync --watch
```

This watches for file changes and syncs them to the cloud automatically. No full redeployment needed.
