---
title: Idempotent message processing with Camel
tags:
  - Camel
category: Software Engineering
date: 2023-09-26 14:48:22
ogimage: cm1.png
excerpt:
  In Enterprise Integration the Idempotent Receiver Pattern ensures that regardless of how many times a message is received the same result is produced. I wanted to see how Camel implements this pattern and explore some of the edge cases.
---


In Enterprise Integration the [Idempotent Receiver Pattern](https://www.enterpriseintegrationpatterns.com/patterns/messaging/IdempotentReceiver.html) ensures that regardless of how many times a message is received the same result is produced. I wanted to see how Camel implements this pattern and explore some of the edge cases.

The setup is below. I have a single queue containing duplicate messages. In order to scale horizontally I want to have multiple receivers but I also want to ensure that the duplicate messages are not processed twice. I'll be using Redis to store the shared state between the two receivers. 

<iframe frameborder="0" style="width:100%;height:413px;" src="https://viewer.diagrams.net/?tags=%7B%7D&highlight=0000ff&edit=_blank&layers=1&nav=1&title=camel-once.drawio#R1Vhdc6IwFP01PHaHEPHjcf3obmdqx10fWh9Tc4V0kDAxKvjrNywXAam1tnWwT%2Bae3CQ351wODhYdLONfikX%2BWHIILMfmsUWHluMQp%2BWYnxRJMqTT7WWApwTHpAKYih0gaCO6FhxWlUQtZaBFVAXnMgxhrisYU0puq2kLGVRPjZgHNWA6Z0EdfRRc%2Bxnade0C%2Fw3C8%2FOTiY0zS5YnI7DyGZfbEkRHFh0oKXU2WsYDCFLycl6ydbdHZveFKQj1exbwnfMwvPvp9m4992XHw9b9wr3BXTYsWOOF%2FwIXK6xYJzkNW19omEZsnsZbI7VF%2B75eBiYiZshWUUb%2BQsRgzuvjtqA0xEfrJXsWTPuAXIJWiUnBBbSLxGHn0JzZbUkHhPySBDnGUHlvv3NBjhkgP2dw1a2xAtz0CoZSaV96MmTBqED7Sq5DnjIytE1U5NxLGSF3L6B1go3P1lpWmYVY6Cdcno5n6fiHi9EwLk0NEwyyOtPi3qbe3EWu1RzeuDM%2BvZopD%2FSpPqpLqSBgWmyqdXy5MMRpVJm9GrPSzOvKGP5V8pSnpUFJzzQslv2PmlTUblJSp%2BZLE8kNUFe6AWdyr82ZOk32P2nGmei3cCbSpDIfcia7MWd6t6KNOhM94kx1pRtwJnJtzuR%2Bm%2F43wQSUMNcGdXZvn%2BxZ%2BsmWxaUTKczJpb%2FIbkVu0jnQMXvmcNWBlPsyPq5u%2B3rUtc9Wt3C8wuRm5blPO97JrnAu0hWtdrNd0as55F1%2FbIDxn3Mt8gsckbarjkhadUckziuW2L6UJeZfNkr8PMKzAaagNsIwcQXvkUNjaXUu9h4xYfENJGvB4ksSHf0D"></iframe>

# The test
I started by creating a unit test to describe the behaviour that I am looking for. Basically, regardless of how many times the same message is submitted I only expect it to be processed once.

```
@Test
void Messages_should_only_be_processed_once() throws Exception {

    Random rand = new Random();
    var id = rand.nextInt(9999);

    receiver.expectedMessageCount(1);
    receiver.expectedBodiesReceived(id);

    camelContext.start();

    producerTemplate.asyncSendBody(endpoint, id);
    producerTemplate.asyncSendBody(endpoint, id);
    producerTemplate.asyncSendBody(endpoint, id);
    producerTemplate.asyncSendBody(endpoint, id);

    receiver.assertIsSatisfied();
}
```

# Idempotent Receiver
Here's the route needed to pass the test. The route is set up to receive asynchronously (up to 10 threads). When a message is received a log message is generated and then we enter the Idempotent Consumer block. We are using Redis to store state and evaluating uniqueness based on the body() of the message. If the message is deemed to be unique it is allowed into the block for processing. If not, it is discarded.

```
from(receiver_queue)
    .routeId(ROUTE_ID)
    .threads(10)
    .log(LoggingLevel.INFO, "Message Received")
    .idempotentConsumer( body(), RedisIdempotentRepository.redisIdempotentRepository(redisTemplate,"some-redis"))
    .process(this::doWork)
    .log(LoggingLevel.INFO, "Contents: ${body}")
    .to(destination_queue)
    .log(LoggingLevel.INFO, "File delivered.");
```

# Edge cases
As is often the case with Camel a simple route hides a fair amount of complexity. In this case the complexity stems from a combination of when items are registered with the idempotentConsumer, when they are removed from the queue and how errors are handled. 

Fortunately Camel offers a fair amount of configurability in how these scenarios are handled. 

On the Idempotent Comsumer front there are a number of options:
 - eager - controls whether the item is registered as soon as it enters the block or on completion of the exchange. Defaults to True.
 - completionEager - controls whether the item is marked as processed on completion of the block or whole exchange. The implication of this is that the item will be marked as complete regardless of the outcome of subsequent actions.
 - skipDuplicate - By default duplicates are rejected however you can opt to process them with an additional header marking that they are duplicate.
 - removeOnFailure - By default items are removed from the Idempotent Consumer on failure of the route. 

On the queuing front the JMS component allows us 2 forms of message acknowledgement: Automatic and Manual. Automatic does what it says on the tin - when the message is received in the route it is marked as received on the queue and removed. In Manual mode the message is only removed on successful completion of the route. If the route fails, the message would not be removed.

How you adjust these settings will have significant impact on the behaviour of your route. Here are some examples:

- If you set eager to false you could receive duplicates if 2 duplicate messages arrive in the queue at the same time and are each picked up by a separate processor. 

-  If completionEager is set and the message fails after the Idempotent Consumer block you need to manually handle retrying the message. A duplicate message later will not be processed as it has already been registered.

- If you are in Manual JMS mode you need to ensure that poison messages are handled otherwise all processing will stop if a message causes an error.

- If you combine Manual JMS mode with completionEager you could create the situation where the item is registered as having been processed with the Idempotent Consumer, but not removed from the queue. On retry the message would be treated as a duplicate as discarded. 

# Conclusion
Idempotent message semantics can be complicated. You need to look at the specific use case, design and performance considerations to figure out the correct way forward. The Camel Idempotent Consumer is effectively providing a de-duping service but as seen above there are edge cases where a duplicate could arise.  You may need to add additional logic to ensure that if a duplicate message does get through that the downstream systems handle it effectively. 





