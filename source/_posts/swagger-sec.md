---
title: Configuring Swagger Security
tags: OpenApi
excerpt: >-
  Having issues with getting Swagger to send an Auth header? Merging multiple
  OpenApi documents at runtime using Node Express? This could help.
date: 2023-05-17 10:35:57
---


I ran into a challenge getting authentication working in Swagger. I wanted people to be able to use the API directly from Swagger. This meant ensuring that Swagger sends a Bearer token with each request. I thought this would be simple, but it took a bunch of searching to get everything working the way I wanted. Here's what I learned....

# The basics
To enable a bearer token in OpenApi 3.0 you need to add a definition for the security schema as well as an indication of where to use the schema.

Here's the security schema declaration:
{% codeblock lang:yaml %}

components:
    schemas:
    securitySchemes:
        BearerAuth:
            type: http
            scheme: bearer

{% endcodeblock %}

Here is how you indicate it should be used. In this case we are saying that we want all operations to use the 'BearerAuth' schema.

{% codeblock lang:yaml %}

security:
    - BearerAuth: []

{% endcodeblock %}

In theory this is all that should have been necessary to get this to work. With this in place you should be able click 'Authorize', provide your Bearer token and use the API.

<img src='swagger1.png'/>

<img src='swagger2.png'/>

This was the theory but in practice Swagger was not sending the Authorization header for any requests.

# The context

We have a Node Express server which implements multiple OpenApi specifications. We are using swagger-jsdoc (6.2.8) and swagger-ui-express (4.6.3) to generate the Swagger interface.

# Combining multiple OpenApi specifications

The issue was that were combining multiple OpenApi documents into a single document at runtime. In the process the information that allowed Swagger to determine which security schema to use was being lost. As a result it assumed all operations were anonymous and so failed to send any Authorization headers. 

The fix was to re-add this information when we created to combined OpenApi document.

{% codeblock lang:javascript %}

const options = { 
    swaggerOptions: {
      url: "/docs/swagger.json", //Link to serve the full OpenApi doc on
    },
    definition: {
        openapi: '3.0.0',
        info: {
          title: 'Inovapp Api documentation',
          version: '1.0.0',
        },
        security: [{
          BearerAuth: [] //Set BearerAuth scheme by default. Note that this scheme is already defined in the OpenApi docs so we are not restating it here.
        }]
      },
    apis: ['./openapi/*.yaml'], //Path to our OpenApi documents.
  };

    const swaggerSpec = swaggerJSDoc(options);  

  app.get("/docs/swagger.json", (req, res) => res.json(swaggerSpec));
  app.use('/docs', swaggerUi.serveFiles(null, options), swaggerUi.setup(null, options));

  {% endcodeblock %}

