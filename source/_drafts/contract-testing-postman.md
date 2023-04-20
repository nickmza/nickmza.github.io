---
title: Contract Testing with Postman
category:
- Software Engineering
tags:
- TDD
- Postman
excerpt:
    I’m giving some training next week on Contract Driven Development using Postman. There’s been a couple of hurdles to overcome to get to a workflow that I feel is efficient so I thought I’d share in the hopes that a) this might help someone else or b) someone could point out a better way to do this!
---

I'm giving some training next week on Contract Driven Development using Postman. There's been a couple of hurdles to overcome to get to a workflow that I feel is efficient so I thought I'd share in the hopes that a) this might help someone else or b) someone could point out a better way to do this. 

# The Scenario

For the training we are using a simplistic banking interface defined using OpenApi. The intention is to show how as a Consumer, Contract-Driven Development helps us to safely move forward whilst another team is still implementing the actual service. 

The service itself is simple. Its an Accounts service with the following endpoints:

| Action | Path | Description |
| ----------- | ----------- | ----------- |
| GET | accounts | Returns the list of accounts for the customer. |
| GET | accounts/{accountId}/transactions | Returns the list of transactions for the specified account.
| POST | accounts | Create a new Account. |

# Creating a failing test
The first thing I'd like to test is the Account creation. As per the OpenApi spec I need to POST the following request:

{% codeblock lang:javascript %}
{
    "AccountType":"Savings Account",
    "Currency": "ZAR"
}
{% endcodeblock %}

and expect the following result:

{% codeblock lang:javascript %}
{
    "AcccountNumber": "61779244",
    "AccountType": "Savings Account",
    "Currency": "ZAR",
    "Balance": 0.00
}
{% endcodeblock %}

I start by setting up this request/response in Postman and pointing it to a Postman Mock server. As there is nothing configured on the Mock Server yet when I run my request I receive:

{% codeblock lang:javascript %}

{
    "error": {
        "name": "mockRequestNotFoundError",
        "message": "Double check your method and the request path and try again.",
        "header": "No matching requests"
    }
}

{% endcodeblock %}

See [here](https://learning.postman.com/docs/designing-and-developing-your-api/mocking-data/setting-up-mock/) for help on configuring a Mock Server with Postman.

# Schema validation

The first thing to get working is the ability to verify that any responses we receive are valid as per the OpenApi specification. Whilst Postman does this out of the box it does not generate a test failure. We need failing tests so that we can run our contract tests as part of the CI/CD pipeline. 

In order to validate responses we need a reference to the OpenApi spec itself. To to this we use the Postman API to download the spec and save it as a variable. There's some gymnastics here as we must first convert our OpenApi document from YAML to JSON which requires an external library - js-yaml. 

You could avoid this by putting the OpenAPI in JSON format directly into the variable but then you'll need to remember to update it when the spec changes.

{% codeblock lang:javascript %}

const schema_url = 'https://api.getpostman.com/apis/API ID/versions/VERSION ID/schemas/SCHEMA ID';

const postRequest = {
  url: schema_url,
  method: 'GET',
  header: {
    'Content-Type': 'application/json',
    'X-Api-Key': pm.variables.get('apiKey')
  }
};

pm.sendRequest("https://cdnjs.cloudflare.com/ajax/libs/js-yaml/4.1.0/js-yaml.min.js", (err, res) => {

   eval(res.text()); //Load the library.

    pm.sendRequest(postRequest, (error, response) => {
        if(!error){
            var schema = response.json().schema.schema;
            var yaml = this.jsyaml.load(schema);
            pm.variables.set('accountsOpenApi', JSON.stringify(yaml,null, 2));
        }
        console.log(error ? error : "Schema Loaded...");
    });

})

{% endcodeblock %}

Now that we have the OpenApi spec we can use this to validate responses. Using the [AJV](https://ajv.js.org/api.html) library we add the OpenApi spec as a schema. We then retrieve a reference to the schema we want to validate against (in this case 'Account') and then validate that our response matches the spec.

{% codeblock lang:javascript %}
var Ajv = require('ajv'),
    ajv = new Ajv({logger: console});

pm.test("Response use a valid schema", function() {

        pm.response.to.have.status(200);
        pm.response.to.be.json;
        const responseJson = pm.response.json();

        var openApi = JSON.parse(pm.variables.get("accountsOpenApi"));
        ajv.addSchema(openApi,"Accounts");

        var accountSchema = ajv.getSchema('Accounts#/components/schemas/Account');
    
        var result = accountSchema(responseJson);

        console.log(accountSchema?.errors);
        var message = accountSchema.errors ? accountSchema.errors[0].dataPath + ' ' +  accountSchema.errors[0].message : '';

        pm.expect(result, message).to.be.true;
});

{% endcodeblock %}

Running this test fails (as expected) because the error response from the Mock Server does not match the requirements of the OpenApi spec. Now let's try get it to pass...

# Getting a simple test to pass
The first thing I'm going to do is configure the Mock Server to return this:

{% codeblock lang:javascript %}
{
    "AcccountNumber": "{ {$randomBankAccount} }"
}
{% endcodeblock %}

It's a little closer to what we want - but it's still not valid. If the schema validation is working this should fail. Note the use of the randomly generated Bank Account number. For more details see [here](https://learning.postman.com/docs/writing-scripts/script-references/variables-list/).

Now when I run the test it fails with the following errors:

{% codeblock lang:javascript %}
    keyword: "required"
    message: "should have required property 'Balance'"
{% endcodeblock %}

The validation seems to be working. Let's update the mock to return a valid response:

{% codeblock lang:javascript %}

{
    "AcccountNumber": "{ {$randomBankAccount} }",
    "AccountType": "{ {$body 'AccountType'} }",
    "Currency": "{ {$body 'Currency'} }",
    "Balance": 0.00
}

{% endcodeblock %}

Success. We now have a passing test! 

At this point I am going to create tests for the negative case as well so that I have a test that proves that validation will fail if the server sends something that does not conform to the spec. I will also implement a test to ensure that the request also conforms to the spec.

I am not specifically going to create tests for every constraint defined in the OpenApi spec. I am going to trust that my colleagues implementing the service will verify their implementation against the spec and hopefully generate a lot of their code as well. The tests that I write from this point will focus on the specific behaviours that my client application relies upon or dynamic scenarios that cannot be easily be captured using OpenApi.

# Behaviour Testing

At this point we have a test that will ensure correct requests and responses as per the OpenApi definition. This is only the end of the beginning though. Whilst OpenApi does a great job of specifying Request and Response formats it is less capable of describing dynamic behaviour. For example, OpenApi can model that a property may contain an array of objects and even specify ranges of values but it cannot easily describe under what conditions these results would occur. For this we need Contract Tests.

In Contract Driven Development we should create tests for the specific behaviour we care about as a Consumer of the service. With these tests in place I can quickly verify if a Producer's implementation of the service agrees to the contract.

Let's add a simple one. Let's say that ZAR-denominated accounts can only be 'Current Accounts'. My expectation then would be that if I submit a request to create a ZAR-denominated Savings Account I should get an 400 error response and an error description.

{% codeblock lang:javascript %}

{
    "AccountType":"Savings Account",
    "Currency": "ZAR"
}

{% endcodeblock %}

{% codeblock lang:javascript %}

{
    "ErrorCode": "Invalid Account Request",
    "Error": "ZAR Denominated Accounts must be Current."
}

{% endcodeblock %}

To achieve this I need to create a Mock for this specific request/response and then add a test to assert the outcome.

{% codeblock lang:javascript %}

pm.test("Ensure ZAR Accounts are Current.", function () {

    pm.response.to.have.status(400);
    pm.response.to.be.json;

    const responseJson = pm.response.json();
    pm.expect(responseJson.ErrorCode).to.contain("Invalid Account Request");

})

{% endcodeblock %}

# Scenario Tests

The above test will handle cases where the expected behaviour can be assessed with a single call to the service. Other behaviours however will require multiple calls to different services. To test these we need to build out scenarios and prompt the Mock server for specific responses. The main risk in creating these types of tests are that then end up being so tightly coupled to specific data and patterns that there is no guarantee that they will pass when used against the real implementation.

Let's take a simple example: I want to ensure that when I create a new account that there should be no transactions. This will require a POST to the 'account' service and then a GET to the account/{account id}/transactions service to verify that there are no transactions.

The challenge is that I already have a Mock for the 'transactions' service which returns an array of transactions. I need a way to tell the Mock server to return an empty array under certain circumstances. 

To implement this using Postman I created a new folder and added the 2 Requests I need to create the account and get the transactions. This allows me to run these tests in sequence and add specific rules.

<img src='scenario.png' width=300/>

I then create a Mock which returns an empty array and call it 'No Results'.

<img src='no-results.png' width=300/>

Finally I add my test to the call to the 'transactions service':

{% codeblock lang:javascript %}

pm.test("There should be no transactions for a new account.", function () {

    pm.response.to.have.status(200);
    pm.response.to.be.json;

    const responseJson = pm.response.json();

    pm.expect(responseJson.length).to.eq(0, "There should be no transactions on a new account.");
})

{% endcodeblock %}

The last thing I need to do is tell the Mock server to use the specific 'No Results' response. To do this I send the 'x-mock-response-name' header with a value of 'No Results' with the call to the 'transactions' service. This asks the Mock server to return this specific results. There are many ways to nudge the Mock server. See [here](https://learning.postman.com/docs/designing-and-developing-your-api/mocking-data/matching-algorithm/) for more details.

This test will pass but it is not a good test. The reason is that there is no guarantee that this will work against the real service. This is because the only reason we receive the empty response is because we asked the Mock server for it. In practice the empty response should be as a consequence of calling the 'transcations' service immediately after creating a new account. At the moment our test does not reflect this. 

Fixing this requires us to ensure that we use the account number returned by the call to 'accounts' as an input to the call to 'transactions'. To do this we save the results for the first call to a variable:

{% codeblock lang:javascript %}

pm.test("Ensure Account Created.", function () {

    pm.response.to.have.status(200);
    pm.response.to.be.json;

    const responseJson = pm.response.json();

    pm.variables.set("newAccountNumber", responseJson.AcccountNumber);
})

{% endcodeblock %}

The we use this variable as an input to the call to 'transactions':

<img src='transactions.png' width=500/>

# Wrapping up 

At this point we can verify that requests and responses are valid as per the OpenApi spec. This ensures that the tests we write have valid requests and that responses, whether from the Mock or real serve, are also valid. We can also write tests to assert for specific behaviour in the context of a single call as well as during a series of calls. What's left is to set-up a build pipeline to run these tests automatically so that both the Consumer and Producer teams can get fast feedback when the implementation breaks the contract.