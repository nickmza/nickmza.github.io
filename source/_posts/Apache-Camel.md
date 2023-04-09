---
title: 'Riding the Camel: A Practical Guide to TDD with Apache Camel'
date: 2023-03-31 13:57:01
tags: 
- TDD
- Camel
category: Software Engineering
excerpt:
    Recently I've been working with a team integrating Fraud Detection software into our Core Banking System (CBS). This involves sending messages asynchronously between the systems. We are using Apache Camel for this as opposed to IBM AppConnect which is our defacto integration platform. One of the things that make Camel attractive is its first-class support for a Test Driven Development approach. Camel allows us to incrementally evolve our integration code using tests to drive the approach. This is very cool. That said, it's not been exactly easy so whilst it's still fresh in my head I thought I would put down some points about how to make this work in your own projects.
---

Recently I've been working with a team integrating Fraud Detection software into our Core Banking System (CBS). This involves sending messages asynchronously between the systems. We are using Apache Camel for this as opposed to IBM AppConnect which is our defacto integration platform. One of the things that make Camel attractive is its first-class support for a Test Driven Development approach. Camel allows us to incrementally evolve our integration code using tests to drive the approach. This is very cool. That said, it's not been exactly easy so whilst it's still fresh in my head I thought I would put down some points about how to make this work in your own projects.

# Scenario
The requirement is quite simple. When specific parts of a Customer's record are modified in the CBS we need to send a message to the Fraud System. The Fraud System exposes a SOAP Interface so we need to translate the XML Format provided by the CBS into a valid SOAP message. In addition we need to handle various retry and reject semantics. For example if the incoming message is badly formatted, and thus not recoverable, we need to send it to a Poison queue. If the Fraud System is temporarily unavailable we need to send it to a Retry queue. At a high level it looks like this:

![A simple diagram of the design](images/camel-overview.png)


The Camel route to achieve this is as follows:

{% codeblock lang:java %}
    from("wmq:queue:PFM.CUSTOMER")
            .routeId("core-banking-pfm-customer-updates")
            .log(LoggingLevel.INFO, "Message Received.")
            .unmarshal().jacksonXml(BatchMultiupdateCustomer.class)
            .process(this::createSoapRequest)
            .setHeader("SOAPAction", constant(SOAP_Update_Action))
            .setHeader(Exchange.CONTENT_TYPE, constant("text/xml; charset=utf-8"))
            .to(pfmConfig.getHost() + "?throwExceptionOnFailure=false")
            .unmarshal().jacksonXml(Envelope.class)
            .process(this::checkResult)
            .log(LoggingLevel.INFO, "Message sent to PFM.");
{% endcodeblock %}


# Setup

In general you follow the same setup procedure as you would for any Springboot test with some additions.

{% codeblock lang:java %}

@SpringBootTest
@CamelSpringBootTest
@UseAdviceWith
@DirtiesContext(classMode = DirtiesContext.ClassMode.AFTER_EACH_TEST_METHOD)
@MockEndpointsAndSkip("https://.*|wmq:.*")
public class CamelRouteTests {

    @Autowired
    protected CamelContext camelContext;

    @Autowired
    protected ProducerTemplate producerTemplate;

}

{% endcodeblock %}



## UseAdviceWith
The 'UseAdviceWith' attribute tells Camel that we are running in test mode and that we will provide 'advice' on how to proceed. If you do not add this attribute Camel will start normally and active all configured routes. See [here](https://www.javadoc.io/static/org.apache.camel/camel-test-spring/2.20.1/org/apache/camel/test/spring/UseAdviceWith.html) for more details.

## DirtiesConext
This attribute tells the test framework to reset the application context after each test invocation. This ensures that the results from one test does not pollute the results of another. See [here](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/annotation/DirtiesContext.html) for more details.

## Camel Context
The Camel Context gives you access to all the routes and services running. This is used to reconfigure routes for testing and gain access to endpoints.

## Producer Template
The Producer Template is used to inject messages into a route.

## Changing the entry-point
In the route above the entry-point for the Route is a message queue. In order to make testing simpler we would want to be able to submit messages directly into the Route rather than having to queue a message. This can be done by instructing Camel to replace parts of the Route for the duration of the tests. For example:

{% codeblock lang:java %}
    AdviceWith.adviceWith(camelContext,
            "core-banking-pfm-customer-updates",
            rb -> rb.replaceFromWith("direct:file:start"));
{% endcodeblock %}

Here we are replacing the original 'From' with  'direct:file:start'. This allows us to post messages to the route using: 

{% codeblock lang:java %}
producerTemplate.sendBody("direct:file:start", request);
{% endcodeblock %}


# Mocks
Mocks allow us to simulate the endpoints of the application so that our tests do not need to connect to real systems. We can make assertions about the mocks to ensure that the system is behaving as expected.

## MockEndpointsAndSkip
To start mocking using Camel the MockEndpointsAndSkip attribute instructs the framework to create a mock for each endpoint that matches the supplied regex expression. In the example above this means creating a mock for endpoints that start with 'https://' or 'wmq://'. 

The 'Skip' suffix here indicates that Camel should not pass on the messages to the underlying endpoint. If this is desirable use the 'MockEndpoints' annotation instead.

## Accessing Mocks

Mocks can be accessed via injection or directly from the Camel Context.

{% codeblock lang:java %}
@EndpointInject("mock:wmq:queue:PFM.POISON")
private MockEndpoint poisonQueue;
{% endcodeblock %}


{% codeblock lang:java %}
var mock = camelContext.getEndpoint("mock:wmq:queue:PFM.POISON");
{% endcodeblock %}

### Mock Naming
Each mock created via MockEndpointsAndSkip or MockEndpoints will create a unique mock with it's own URI. The format of this URI is the original URI prefixed with 'mock'.

For example:

| URI | Mock URI |
| ----------- | ----------- |
|https://example.com/Fraud/DetectionService.svc|mock:https:example.com/Fraud/DetectionService.svc|
|wmq:queue:PFM.POISON|mock:wmq:queue:PFM.POISON|

**Note:** Notice that for the HTTP endpoint the forward slashes are removed from the mock name. If you have any doubt Camel outputs the URI of both the original endpoint and the mock endpoint in the logs.

## Asserting Mock Behaviour
Once you have a reference to the mock you can set your expectations and then make an assertion in the test.

{% codeblock lang:java %}
@Test
public void validMessageShouldBeProcessed() throws Exception {

    var request = createSampleRequest();

    replaceFromWithMock(); //Replace the From with 'direct:file:start'

    camelContext.start(); //Start Camel

    pfmEndpoint.expectedMessageCount(1);

    producerTemplate.sendBody("direct:file:start", request); //Send message to the Route...

    pfmEndpoint.assertIsSatisfied();
}
{% endcodeblock %}

In this example we expect the pfmEndpoint queue to have a single message on completion of the test.

## Returning Values from a Mock

In some cases you need to be able to control the specific response from a mock. In these scenarios you can use AdviceWith to replace the original end-point with one that models the specific behaviour you require. For example:

{% codeblock lang:java %}

var faultContent = faultResultResource.getContentAsString(StandardCharsets.UTF_8);

var header = new SimpleExpression("500");
header.setResultType(Integer.class);

AdviceWith.adviceWith(camelContext,
        "core-banking-pfm-customer-updates",
        rb -> rb.weaveByToUri(pfmConfig.getHost())
                .replace()
                .setHeader(CAMEL_HTTP_RESPONSE_CODE, header)
                .setBody(new ConstantExpression(faultContent)));

{% endcodeblock %}

Here we are configuring the endpoint to return an HTTP 500 return code and a specific fault payload.

Here is the full test:
{% codeblock lang:java %}

@Test
public void parseFaultResult() throws Exception {

    var request = createSampleRequest();

    var faultContent = faultResultResource.getContentAsString(StandardCharsets.UTF_8);

    replaceFromWithMock();

    var HTTPHeader = new SimpleExpression("400");
    HTTPHeader.setResultType(Integer.class);

    AdviceWith.adviceWith(camelContext,
            "core-banking-pfm-customer-updates",
            rb -> rb.weaveByToUri(pfmConfig.getHost()+"?throwExceptionOnFailure=false")
                    .replace()
                    .setHeader(CAMEL_HTTP_RESPONSE_CODE, HTTPHeader)
                    .setBody(new ConstantExpression(faultContent)));

    poisonQueue.expectedBodiesReceived(faultContent);
    poisonQueue.expectedMessageCount(1);
    retryQueue.expectedMessageCount(0);

    camelContext.start();

    producerTemplate.sendBody("direct:file:start", request);

    poisonQueue.assertIsSatisfied();
    retryQueue.assertIsSatisfied();
}

{% endcodeblock %}




