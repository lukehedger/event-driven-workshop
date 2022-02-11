---
coverY: 0
---

# Event-Driven Architecture Workshop

## Welcome to the workshop 👋

Welcome to the Event-Driven Architecture workshop! In this workshop we will build an event-driven application for sending Christmas wishlist letters to Santa Claus.

Read the [workshop announcement post](https://matcha.level-out.com/posts/event-driven-workshop/) for a definition of event-driven architecture.

{% hint style="info" %}
This workshop was created by [Luke Hedger](https://twitter.com/level\_out/) and run for the first time with a [LEGO Engineering](https://twitter.com/LEGOEngineering) team in December 2021.
{% endhint %}

### Letters to Santa 🎅

Send your Christmas list to Santa! You can ask for any LEGO set or a surprise from Santa's workshop! Santa is also keen to keep costs low this year, so doesn't want to use any Lambda functions.

Santa's elves open the wishlists sent to Santa and check everything looks legit! Santa has a central event bus for receiving gift requests. The elves can forward gifts on a wishlist to the event bus. Santa will then pick up the gift requests on Christmas Eve.
