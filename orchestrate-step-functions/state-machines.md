---
description: Add some state machines to process events
---

# State Machines

Next, we are going to add something to consume the events we are sending to our event bus!

We will use AWS Step Functions to process our wishlist events. We will have one workflow for processing LEGO gift requests and one for processing surprise gift requests.

We will use [Express Workflows](https://docs.aws.amazon.com/step-functions/latest/dg/bp-express.html) as these are better suited to event processing than Standard Workflows.

### Add state machine for LEGO gifts

Wishlists that include a LEGO gift need to be sent to the LEGO Event-Driven Elves 🧝‍♂️, via another EventBridge event bus. You will need to ask the Event-Driven Elves for the central event bus ARN. And don't forget to append your _elf source_ (e.g. `elf-luke`) to the event you send so the elves know where the gift request has come from:

```typescript
import {
  StateMachine,
  StateMachineType,
  TaskInput,
} from "aws-cdk-lib/aws-stepfunctions";

const legoGiftStateMachine = new StateMachine(
  this,
  "EDAWorkshopLegoGiftStateMachine-YOUR_USER_NAME",
  {
    definition: new EventBridgePutEvents(this, "PutLegoGiftEvent", {
      entries: [
        {
          eventBus: EventBus.fromEventBusArn(
            this,
            "EDAWorkshopCentralBus-YOUR_USER_NAME",
            "CENTRAL_EVENT_BUS_ARN"
          ),
          detail: TaskInput.fromObject({
            legoSet: JsonPath.stringAt("$.legoSKU"),
            giftTo: JsonPath.stringAt("$.from"),
          }),
          detailType: "GiftRequested",
          source: "elf-YOUR_NAME",
        },
      ],
    }),
    stateMachineName: "EDAWorkshopLegoGiftStateMachine-YOUR_USER_NAME",
    stateMachineType: StateMachineType.EXPRESS,
  }
);
```

{% hint style="success" %}
<mark style="color:yellow;">**Bonus:**</mark> Add a nice message to the elves as part of the event detail, like: "Hey LEGO Event-Driven Elves, {from} would like LEGO set {legoSKU}. Thanks!"
{% endhint %}

### Add state machine for surprise gifts

Wishlists that include a surprise gift can to be sent direct to the workshop, ready for random gift selection, via an S3 bucket. You can ask the Event-Driven Elves for the central bucket ARN too. Again, don't forget to append your elf source to the object!

```typescript
import { Bucket } from "aws-cdk-lib/aws-s3";
import {
  JsonPath,
  StateMachine,
  StateMachineType,
  TaskInput,
} from "aws-cdk-lib/aws-stepfunctions";
import {
  CallAwsService,
  EventBridgePutEvents,
} from "aws-cdk-lib/aws-stepfunctions-tasks";

const centralBucket = Bucket.fromBucketArn(
  this,
  "EDAWorkshopCentralBucket-YOUR_USER_NAME",
  "CENTRAL_BUCKET_ARN"
);

const surpriseGiftStateMachine = new StateMachine(
  this,
  "EDAWorkshopSurpriseGiftStateMachine-YOUR_USER_NAME",
  {
    definition: new CallAwsService(this, "PutSurpriseGiftObject", {
      action: "PutObject",
      iamAction: "s3:*",
      iamResources: [centralBucket.arnForObjects("*")],
      parameters: {
        Body: JSON.stringify({ giftTo: JsonPath.stringAt("$.from") }),
        Bucket: centralBucket.bucketName,
        Key: JsonPath.stringAt("$.from"),
        Tagging: "source=elf-YOUR_NAME",
      },
      service: "s3",
    }),
    stateMachineName: "EDAWorkshopSurpriseGiftStateMachine-YOUR_USER_NAME",
    stateMachineType: StateMachineType.EXPRESS,
  }
);

surpriseGiftStateMachine.addToRolePolicy(
  new PolicyStatement({
    actions: ["s3:*"],
    resources: [centralBucket.bucketArn],
  })
);
```

### Add event rules

Now we have our Step Functions workflows, we need to add rules to our event bus, per gift type, to execute the state machines based on the gift requested.

```typescript
import {
  EventField,
  EventBus,
  Rule,
  RuleTargetInput,
} from "aws-cdk-lib/aws-events";
import { SfnStateMachine } from "aws-cdk-lib/aws-events-targets";

new Rule(this, "EDAWorkshopLegoGiftRule-YOUR_USER_NAME", {
  eventBus: eventBus,
  eventPattern: {
    detail: {
      gift: ["lego"],
    },
    detailType: ["WishlistReceived"],
    source: ["workshop.eda"],
  },
  ruleName: "EDAWorkshopLegoGiftRule-YOUR_USER_NAME",
  targets: [
    new SfnStateMachine(legoGiftStateMachine, {
      input: RuleTargetInput.fromObject({
        from: EventField.fromPath("$.detail.from"),
        legoSKU: EventField.fromPath("$.detail.legoSKU"),
      }),
    }),
  ],
});

new Rule(this, "EDAWorkshopSurpriseGiftRule-YOUR_USER_NAME", {
  eventBus: eventBus,
  eventPattern: {
    detail: {
      gift: ["surprise"],
    },
    detailType: ["WishlistReceived"],
    source: ["workshop.eda"],
  },
  ruleName: "EDAWorkshopSurpriseGiftRule-YOUR_USER_NAME",
  targets: [
    new SfnStateMachine(surpriseGiftStateMachine, {
      input: RuleTargetInput.fromObject({
        from: EventField.fromPath("$.detail.from"),
      }),
    }),
  ],
});
```

Deploy and test one more time - check that both types of gift are processed correctly and end up where they should!

Congratulations - you've saved Christmas (and built a neat event-driven architecture on AWS!) Nice work&#x20;

Here are a few more bonuses...

{% hint style="success" %}
<mark style="color:yellow;">**Bonus:**</mark> Enable X-Ray tracing on your state machines
{% endhint %}

{% hint style="success" %}
<mark style="color:yellow;">**Bonus:**</mark> Setup the [EventBridge websocket](https://github.com/boyney123/cdk-eventbridge-socket) stack for monitoring via Postman
{% endhint %}

{% hint style="success" %}
<mark style="color:yellow;">**Bonus:**</mark> Select a random gift from available surprises to help Santa out. You'll need to store the list of surprise gifts somewhere.
{% endhint %}

{% hint style="success" %}
<mark style="color:yellow;">**Bonus:**</mark> Only allow 3 gifts per person - gifts don't grow on trees!
{% endhint %}
