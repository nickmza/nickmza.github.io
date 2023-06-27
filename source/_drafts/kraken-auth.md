---
title: Authenticate the KrakenD!
tags:
  - devops
category:
  - Release The KrakenD
ogimage: ka2.png
excerpt:
    In a previous article I described getting a minimal KrakenD installation up and running using a simple Node Backend. I also configured logging and tracing so that I could see what was happening. Today I'm going to explore some of KrakenD's authentication capabilities.    
---

In a [previous article](https://nickmck.net/2023/06/23/release-the-krakend/) I described getting a minimal KrakenD installation up and running using a simple Node Backend. I also configured logging and tracing so that I could see what was happening. Today I'm going to explore some of KrakenD's authentication capabilities.

## Setting up JWT Authentication
When you define an endpoint in KrakenD there are a number of options for controlling how requests are authenticated.

<img src="ka1.png"/>

I want to use KrakenD to verify incoming JWT tokens. There are a number of advantages to this:
- By validating tokens early we ensure only valid, authenticated and authorized requests reach the backend service in the first place.
- Load on the back-end is reduced.
- Token Management and configuration is centralised.

> Note this does not eliminate the need for backend service validation entirely. The API Gateway acts as a first line of defense, but the backend should still enforce its own security measures and validate requests for complete protection.

 Setting this up was *mostly* straightforward. The first thing I needed to do was to expose my keys as a [JWKS](https://openid.net/specs/draft-jones-json-web-key-03.html) endpoint. I did this using the [node-jose](https://github.com/cisco/node-jose) library for Node. If you're interested in the details of this let me know in the comments.

Now when I hit my JWKS endpoint I get this:

{% codeblock lang:json %}

{
    "keys": [
    {
    "kty": "RSA",
    "kid": "DATA",
    "use": "sig",
    "alg": "RS256",
    "e": "DATA",
    "n": "DATA"
    }]
}

{% endcodeblock %}

Next step is to connect KrakenD to my JWKS Endpoint. This is done by adding an auth/validator to the Endpoint's extra_config section:

{% codeblock lang:json %}

      "extra_config": {
        "auth/validator": {
          "alg": "RS256",
          "jwk_url": "https://MyEndPoint/jwks",
          "operation_debug": true
        }
      }

{% endcodeblock %}

That's it. Incoming requests that provide a token are authenticated using the information in the JWKS.

There was one snag. I was receiving this error:

```
▶ ERROR [ENDPOINT: /v1/atms][JWTValidator] Unable to validate the token: algorithm is invalid
```

> If you run into trouble set the operation_debug property to true on the auth/validator.

This was curious because my tracing showed that my JWKS endpoint was not being hit at all. It turns out that this error can also happen if there are connectivity issues. The issue was that my containers were not talking to each other. I fixed this and all was fine.

## Parameter Forwarding

<img src="ka3.png" width="500px"/>

By default KrakenD will not automatically forward the Authorization header to the downstream system. To do this add 'Authorization' to the input_headers array of the endpoint.

## Authentication Options
KrakenD allows a lot of flexibility in terms of how tokens are processed. You can test for specific Scopes, the Issuer, the Audience and specific roles. In addition you can cache results to improve performance for repeated validations. This is all done via the auth/validator object. For example to constrain the audience to 'https://www.mcb.mu/api' and add caching:

{% codeblock lang:json %}

      "extra_config": {
        "auth/validator": {
          "alg": "RS256",
          "jwk_url": "https://MyEndPoint/jwks",
          "audience": "https://www.mcb.mu/api",
          "cache": true
        }
      }

{% endcodeblock %}

## Claim Extraction
KrakenD can extract claims from the token and use them in the requests made to downstream services. For example I would like to take my Subject claim and inject it as a HTTP Header with the key 'x-user'.

This can be achieved via the propagate_claims property of the auth/validator.

{% codeblock lang:json %}

      "extra_config": {
        "auth/validator": {
          "propagate_claims": [
            ["sub", "x-user"]
          ]
        }
      }

{% endcodeblock %}

> Note you will also have to add the x-user header to the input_headers array otherwise KrakenD will block it.

Here's the result. Note the 'x-user' header in the debug trace.

<img src="ka4.png"/>

## Debugging

If it's not working start by [decoding](https://jwt.io/) your token and ensuring that the claims and expiry dates are valid. 

<img src="ka2.png">

If that does not work KrakenD provides a __debug endpoint to help you figure out what's going on. To use this:
- Set the debug_endpoint to 'true'.
- Set your log level to 'DEBUG'. 
- Add a new backend that points to KrakenD as the Host and '/__debug' as the URL. 

With this in place KrakenD will log everything that is sent to the __debug endpoint so that you can see if anything is being filtered out or mismatched.

## Next Steps
The next thing I'd like to play with is seeing how I can shape the request and response messages as they pass through the gateway. For example seeing if I can get the gateway to filter out specific properties of fields with specific patterns. 


