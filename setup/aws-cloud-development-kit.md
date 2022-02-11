---
description: Create a new CDK project
---

# AWS Cloud Development Kit

## What is the AWS CDK?

We will use the AWS Cloud Development Kit (or, CDK) to define the cloud infrastructure for our event-driven architecture _"as code"_. You can learn more about the AWS CDK from these resources:

- [https://aws.amazon.com/cdk/](https://aws.amazon.com/cdk/)
- [https://docs.aws.amazon.com/cdk/latest/guide/getting_started.html](https://docs.aws.amazon.com/cdk/latest/guide/getting_started.html)
- [https://cdkworkshop.com/](https://cdkworkshop.com)
- [https://explore.skillbuilder.aws/learn/course/internal/view/elearning/1475/aws-cloud-development-kit-primer](https://explore.skillbuilder.aws/learn/course/internal/view/elearning/1475/aws-cloud-development-kit-primer)

## `cdk init`

Run this command to initialise a CDK project that uses TypeScript:

```shell
npx cdk init app --language typescript
```

### Project structure

Take some time to look at the folders and files that have been created for you.

**CDK Stacks**

Stacks are collections of resources that get transformed into CloudFormation templates and can then be deployed to AWS as CloudFormation Stacks. These live in the `/lib` directory.

**CDK Apps**

Apps are collections of one or more stacks. These live in the `/bin` directory.

### Commands

Take a look at the commands available to you in the `scripts` section of the `package.json` file. We will use these later to build and deploy our application.

### Customise

Let's customise the boilerplate CDK project. You can go ahead and rename the "example" stack that has been created for you to "eda-workshop" (or, whatever you like!). You'll need to rename it in multiple files and filenames - check the following places: `/bin`, `/lib`, `/test`, `cdk.json`, `package.json`.

You can then also configure the CDK app to use the default account/region from the AWS profile you setup earlier - do this in the `bin/eda-workshop.ts` file. You should also **rename the stack with your user name**, so it can be identified when deploying to an AWS account that is used by multiple users.

```typescript
#!/usr/bin/env node
import "source-map-support/register";
import { App } from "aws-cdk-lib";
import { EDAWorkshopStack } from "../lib/eda-workshop-stack";

const app = new App();

new EDAWorkshopStack(app, "EDAWorkshopStack-{YOUR_USER_NAME}", {
  env: {
    account: process.env.CDK_DEFAULT_ACCOUNT,
    region: process.env.CDK_DEFAULT_REGION,
  },
});
```
