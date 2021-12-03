---
cover: >-
  https://images.unsplash.com/photo-1526304640581-d334cdbbf45e?ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&ixlib=rb-1.2.1&auto=format&fit=crop&w=2970&q=80
coverY: 0
description: Add an API Gateway REST API
---

# REST API

We will add an API Gateway REST API with a one endpoint:

```http
POST /dear-santa { gift: "lego" | "surprise", legoSKU?: "40499", from: "emmet" }
```

### Install packages

```shell
npm install --save-dev --save-exact @aws-cdk/aws-apigateway
```

### Update CDK stack

Edit the `lib/eda-workshop-stack.ts` file to look like this - adding your user name to the API name to identify it:

```typescript
import { Construct, Stack, StackProps } from "@aws-cdk/core";
import { RestApi } from "@aws-cdk/aws-apigateway";

export class EDAWorkshopStack extends Stack {
  constructor(scope: Construct, id: string, props?: StackProps) {
    super(scope, id, props);

    new RestApi(this, "EDAWorkshopAPI-YOUR_USER_NAME");
  }
}
```

### Run a build

Take a look at the JavaScript files that get created:

```
npm run build
```

### Synthesise the CloudFormation template

Take a look at the `cdk.out/EDAWorkshopStack.template.json` file that gets created, this is the CloudFormation template that will be deployed:

```
npm run cdk synth
```

{% hint style="success" %}
<mark style="color:yellow;">**Bonus:**</mark> Add a simple test for your REST API to the `eda-workshop.test.ts` file and run `npm test` to execute it
{% endhint %}

### Deploy CDK app

Now you can deploy your new CDK app (complete with an API!) to AWS - whenever you want to deploy your app just run this command and it will build and synthesise for you:

```
npm run cdk deploy
```

### Test API endpoint

Once the CDK app has been deployed, you should see an API Gateway URL in the terminal. Use it to test out your API response:

```shell
curl -X POST https://{YOUR_API_URL}.amazonaws.com/prod/dear-santa
```

You should get an "internal server error" - this is because we haven't got anything behind the endpoint yet.

So... let's put something behind the endpoint!

{% hint style="success" %}
<mark style="color:yellow;">**Bonus:**</mark> Enable AWS X-Ray tracing for your REST API
{% endhint %}
