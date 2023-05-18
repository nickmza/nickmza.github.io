---
title: Unit Testing Camel - A simple example
tags:
  - TDD
  - Camel
excerpt: >-
  In a previous postI wrote about taking a Test-Driven approach to Camel
  development. I've been asked to share the code and explain some of the steps
  in more detail so here we go...
date: 2023-05-18 18:10:04
---


In a [previous post](https://nickmck.net/2023/03/31/Apache-Camel/) I wrote about taking a Test-Driven approach to Camel development. I've been asked to share the code and explain some of the steps in more detail so here we go...

> The code for this article is [here](https://github.com/nickmza/camel-tdd.git)

# The Route

For this example I've created a simple route:

{% codeblock lang:java %}

        from("timer://example?repeatCount=1")
                .routeId("simple-rest")
                .process(this::createMessage)
                .marshal().json()
                .setHeader(Exchange.CONTENT_TYPE, constant("text/xml; charset=utf-8"))
                .to("https://en1gvb5qo7vg4.x.pipedream.net");

{% endcodeblock %}

This code is triggered from a Timer, the 'repeatCount' option specifies that it should fire only once. We then create a message in the createMessage function, convert the message to JSON, set the 'Content-Type' header and send the message to an HTTP endpoint. In this case im using a [RequestBin](https://public.requestbin.com/).

If we run the application we can see this in the RequestBin:

<img src="./camel1.png" />

# Mocking the Http Endpoint
To effectively test this route we must be able to mock the HTTP endpoint. To do this we need to add the following annotation to our test class:

{% codeblock lang:java %}

@MockEndpointsAndSkip("https://.*")
public class simpleRestRouteTests {
}

{% endcodeblock %}

This tells the test framework to create a Mock for any endpoint that starts with 'https://'. This Mock can then be accessed like this:

{% codeblock lang:java %}

@EndpointInject("mock:https:en1gvb5qo7vg4.x.pipedream.net")
private MockEndpoint restEndpoint;

{% endcodeblock %}

> The value passed to EndpointInject must match the endpoint value configured in your route. 

# A Simple Test

Let's write a test to ensure that Camel is calling the endpoint. 

The first thing we want to do is to replace the Timer component that triggers the flow. This will make it easier to inject data into the Route. To do this we use AdviceWith.

{% codeblock lang:java %}

    AdviceWith.adviceWith(camelContext,
            "simple-rest",
            rb -> rb.replaceFromWith("direct:file:start"));

{% endcodeblock %}

This swops the 'From' in the Route (currently a Timer) with a new Direct component. 

Next we can set up an assertion. We want to configure that the Http endpoint is called exactly once. We can do this via the Mock we referenced earlier: 

{% codeblock lang:java %}

restEndpoint.expectedMessageCount(1);

{% endcodeblock %}

Finally we want to send a message into the route and assert that the Mock was called:


{% codeblock lang:java %}

producerTemplate.sendBody("direct:file:start", "Hello from Test!");
restEndpoint.assertIsSatisfied();

{% endcodeblock %}

Here's the full listing...

{% codeblock lang:java %}

    @Test
    public void EnsureThatEndpointIsCalled() throws Exception {

        AdviceWith.adviceWith(camelContext,
                "simple-rest",
                rb -> rb.replaceFromWith("direct:file:start"));

        camelContext.start();

        restEndpoint.expectedMessageCount(1);

        restEndpoint.whenAnyExchangeReceived( (Exchange exchange) -> {
            logger.log(Level.INFO, "Mock was HIT!!!");
        });

        producerTemplate.sendBody("direct:file:start", "Hello from Test!");

        restEndpoint.assertIsSatisfied();
    }

{% endcodeblock %}

# Sending specific results from the Mock
Sometimes we need a Mock to return a specific response body, header or HTTP response code. To set this up we need to use AdviceWith again.

{% codeblock lang:java %}

var HTTPHeader = new SimpleExpression("200");
HTTPHeader.setResultType(Integer.class);

AdviceWith.adviceWith(camelContext,
        "simple-rest",
        rb -> rb.weaveByToUri("https://en1gvb5qo7vg4.x.pipedream.net")
                .replace()
                .setHeader("CamelHttpResponseCode", HTTPHeader)
                .setBody(new ConstantExpression("Response")));

{% endcodeblock %}

What's happening here is that we are replacing the endpoint "https://en1gvb5qo7vg4.x.pipedream.net" with a new implementation that returns our custom HTTP header and a body of "Response".

# Next Steps
This should be enough to get some basic testing in place. Let me know in the comments if there are any other scenarios you'd like to see covered.