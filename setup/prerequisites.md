---
description: Setup your environment with some prerequisites
---

# Prerequisites

## Install Node.js

[Install Node.js](https://nodejs.dev) if you don't already have it on your machine.

{% hint style="info" %}
**Good to know:** Use a Node version manager (like [`nvm`](https://github.com/nvm-sh/nvm)`)` to easily switch between different versions of Node.js
{% endhint %}

## Install AWS CLI

Install the lastest version (v2) of the [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) if you don't already have it on your machine.

### Configure your AWS CLI profile

Run the following command to setup AWS credentials that can be used by the AWS CLI tool:

```shell
aws configure
```

