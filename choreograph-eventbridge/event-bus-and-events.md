---
description: Add an event bus and start sending events to it
---

# Event Bus and Events

We are going to add an integration to our API Gateway REST API that will send events to our EventBridge event bus whenever the `POST /dear-santa` endpoint receives an HTTP POST request.

### Install packages

```
npm install --save-dev --save-exact @aws-cdk/aws-events
```

### Add EventBridge event bus

Update your CDK stack with an event bus:

```typescript
import { Construct, Stack, StackProps } from "@aws-cdk/core";
import { RestApi } from "@aws-cdk/aws-apigateway";
import { EventBus } from "@aws-cdk/aws-events";

export class EDAWorkshopStack extends Stack {
  constructor(scope: Construct, id: string, props?: StackProps) {
    super(scope, id, props);

    const eventBus = new EventBus(this, "EDAWorkshopBus-YOUR_USER_NAME", {
      eventBusName: "EDAWorkshopBus-YOUR_USER_NAME",
    });

    const api = new RestApi(this, "EDAWorkshopAPI-YOUR_USER_NAME");

    const dearSanta = api.root.addResource("dear-santa");

    dearSanta.addMethod("POST");
  }
}
```

{% hint style="success" %}
<mark style="color:yellow;">**Bonus:**</mark> Add a CloudWatch Log Group target to your Event Bus to forward all events as logs to CloudWatch
{% endhint %}

### Permissions

Next, we need to give our API permission to put events on the bus. Install the IAM CDK package:

```
npm install --save-dev --save-exact @aws-cdk/aws-iam
```

Then add this to your stack:

```typescript
const apiRole = new Role(this, "EDAWorkshopAPIRole-YOUR_USER_NAME", {
  assumedBy: new ServicePrincipal("apigateway"),
  inlinePolicies: {
    putEvents: new PolicyDocument({
      statements: [
        new PolicyStatement({
          actions: ["events:PutEvents"],
          resources: [eventBus.eventBusArn],
        }),
      ],
    }),
  },
});
```

Now we can update our REST API with an [API integration](https://docs.aws.amazon.com/apigateway/latest/developerguide/how-to-integration-settings.html). In our case, this will be a direct integration between API Gateway and EventBridge - any requests sent to the `POST /dear-santa` endpoint will put an event on the bus. To do this, we need to add some more imports and update the `dearSanta` method:

```typescript
import { Integration, IntegrationType, RestApi } from "@aws-cdk/aws-apigateway";

dearSanta.addMethod(
  "POST",
  new Integration({
    type: IntegrationType.AWS,
    uri: `arn:aws:apigateway:${Aws.REGION}:events:path//`,
    integrationHttpMethod: "POST",
    options: {
      credentialsRole: apiRole,
      requestParameters: {
        "integration.request.header.X-Amz-Target": "'AWSEvents.PutEvents'",
        "integration.request.header.Content-Type":
          "'application/x-amz-json-1.1'",
      },
      requestTemplates: {
        "application/json": JSON.stringify({
          Entries: [
            {
              Detail: "$util.escapeJavaScript($input.body)",
              DetailType: "WishlistReceived",
              EventBusName: eventBus.eventBusName,
              Source: "workshop.eda",
            },
          ],
        }),
      },
      integrationResponses: [
        {
          statusCode: "200",
          responseTemplates: {
            "application/json": "",
          },
        },
      ],
    },
  }),
  { methodResponses: [{ statusCode: "200" }] }
);
```

Take some time to look through this code. Note that any [method reponses](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-method-settings-method-response.html) must map to  a corresponding [integration response](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-integration-settings-integration-response.html).

### Deploy CDK app

```
npm run cdk deploy
```

### Test API endpoint

```shell
curl -X POST https://{YOUR_API_URL}.amazonaws.com/prod/dear-santa
```

You should now get a successful response from EventBridge to say it has received an event. It'll look something like this:

```
{"Entries":[{"EventId":"some-uuid"}],"FailedEntryCount":0}
```

{% hint style="success" %}
<mark style="color:yellow;">**Bonus:**</mark> Enable AWS X-Ray tracing on your event bus
{% endhint %}

### Validate API requests

Going back to the code you added to the stack, notice this line in the request template`"Detail": "$util.escapeJavaScript($input.body)"` - here we are passing the entire API request body to EventBridge. We need to make sure the request is valid before sending the data to EventBridge.

API Gateway allows us to add [request validation](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-method-request-validation.html) via JSON Schema.

We need two resources to enforce request body validation - a **request validator** and a **JSON Schema-based model**:

```typescript
import {
  Integration,
  IntegrationType,
  JsonSchemaType,
  JsonSchemaVersion,
  Model,
  RequestValidator,
  RestApi,
} from "@aws-cdk/aws-apigateway";

const dearSantaRequestValidator = new RequestValidator(
  this,
  "EDAWorkshopRequestValidator-YOUR_USER_NAME",
  {
    restApi: api,
    requestValidatorName: "EDAWorkshopRequestValidator-YOUR_USER_NAME",
    validateRequestBody: true,
  }
);

const dearSantaRequestModel = new Model(
  this,
  "EDAWorkshopRequestModel-YOUR_USER_NAME",
  {
    contentType: "application/json",
    modelName: "WishlistRequestModelYOUR_USER_NAME",
    restApi: api,
    schema: {
      schema: JsonSchemaVersion.DRAFT7,
      title: "Wishlist",
      type: JsonSchemaType.OBJECT,
      properties: {
        from: {
          description: "Name of wishlist creator",
          type: JsonSchemaType.STRING,
        },
        gift: {
          description: "Type of gift on the wishlist",
          type: JsonSchemaType.STRING,
          enum: ["lego", "surprise"],
        },
        legoSKU: {
          description: "LEGO product ID",
          type: JsonSchemaType.STRING,
        },
      },
      required: ["from", "gift"],
    },
  }
);
```

These resources are then used in our `dearSanta` method's options:

```typescript
{
  methodResponses: [{ statusCode: "200" }],
  requestModels: { "application/json": dearSantaRequestModel },
  requestValidator: dearSantaRequestValidator,
}
```

### Deploy again

```
npm run cdk deploy
```

### Test again

```shell
curl -X POST https://{YOUR_API_URL}.amazonaws.com/prod/dear-santa
```

The request should be rejected as it failed the validation you just added. Try updating your curl request with the expected body. You should receive the successful response from EventBridge!

{% hint style="success" %}
<mark style="color:yellow;">**Bonus:**</mark> Update the model to only accept "lego" gift requests if a legoSKU is provided
{% endhint %}
