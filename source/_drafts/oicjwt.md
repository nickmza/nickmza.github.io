---
title: JWT User Assertion with Oracle Integration Cloud
tags:
---

Nothing brings out the blogger in me like spending a couple of days wrestling with a problem. This weeks' challenge has been configuring OAuth 2 User Assertion on the Oracle Integration Cloud (OIC). It's taken a couple of days of trial and error but it's finally working! So to hopefully save future generations some time and frustration here's how it's done...

# The scenario
We want to push data from our O365 Sharepoint environment into our Oracle ERP system via a Power Automate Flow. We have used Oracle Integration Cloud to create a REST Trigger that receives the payload and processes it. We want to use the JWT User Assertion flow so that the client does not need to have access to any credentials.

# About the JWT User Assertion Flow

The OAuth 2.0 "JWT Assertion Grant" or "JWT Bearer Token Grant" allows a client to use a JWT as an assertion to obtain an access token from an authorization server.

Here's a basic overview of the flow:

1. Client Creates JWT Assertion:
    - The client creates a JWT (JSON Web Token) containing relevant information, such as its identity, requested scope, and any other necessary claims.

2. Client Sends Assertion to Authorization Server:
    - The client sends the JWT as an assertion to the authorization server, typically in the assertion parameter of the token request.

3. Authorization Server Validates JWT:
    - The authorization server validates the JWT, checking its signature, expiration, and any other relevant claims.

4. Token Issuance:
    - If the JWT is valid, the authorization server responds with an access token.

5. Client Uses Access Token:
    - The client can now use the obtained access token to access protected resources on behalf of the user.

This flow is useful in scenarios where the client can provide a signed assertion (JWT) to prove its identity and request access without going through the typical user authorization process involving redirects and user interactions.

# Create and export your certificate

Generate the self-signed key pair
```
keytool -genkey -keyalg RSA -alias assert -keystore sampleKeystore.jks -storepass samplePasswd -validity 365 -keysize 2048
```

Export the public key for signing the JWT assertion

```
keytool -exportcert -alias assert -file assert.cer -keystore sampleKeystore.jks -storepass samplePasswd
```

Convert the keystore to P12 format
```
keytool -importkeystore -srckeystore sampleKeystore.jks -srcstorepass samplePasswd -srckeypass samplePasswd -srcalias assert -destalias assert -destkeystore assert.p12 -deststoretype PKCS12 -deststorepass samplePasswd -destkeypass samplePasswd
```

Export the private key from the P12 keystore
```
openssl pkcs12 -in assert.p12 -nodes -nocerts -out private_key.pem
```

# Create your Client Application

From the Oracle Identity Cloud Applications panel choose 'Add'.
<img src="jwt1.png"/>

Choose 'Confidential Application'
<img src="jwt2.png"/>

Under the 'Client Configuration' set the 'Allowed Grant Types' to 'Client Credentials' and 'JWT Assertion'.
<img src="jwt3.png"/>

Under 'Security' choose 'Trusted Client' and upload the .cer file you created earlier. Set the alias for the certificate and keep track of this value as you will need it later.
<img src="jwt4.png"/>

<img src="jwt5.png"/>

Under 'Token Issuance Policy' select 'Specific' and then 'Add Scope' under the 'Resources' section.

<img src="jwt6.png"/>

Find the OIC application that you want to provide access to and click the '>' symbol.

<img src="jwt7.png"/>

Select the scope that ends in 'urn:opc:resource:consumer::all'. Keep track of this value as you will need it later.

<img src="jwt8.png"/>

You can accept the defaults for the remaining steps and Save your application. Don't forget to Activate it afterwards. Take note of the Client Id and Secret.

<img src="jwt10.png"/>

# Create Trusted Partner
Even though we added the certificate to the application we also need to add it as a Truster Partner. In Oracle Identity Cloud navigate to Settings, Partner Settings and click 'Import' under 'Partner Settings'.
<img src="jwt9.png"/>

Upload the .cer file you created earlier and set the alias to the same value you used when creating the application.

# Generate the JWT User Assertion

You now need to create and sign a JWT token that describes the user and scope you want access to. You then submit this token to Oracle to retrieve a Bearer token that can be used to access your OIC resource.

The relevant fields in the JWT Payload are:

| Field | Description |
| sub | the user name for whom user assertion is generated. |
| aud | https://identity.oracle.com/ |
| kid | The id of the key to use to verify the signature. Set this to the alias you used when uploading the certificate. |
| iss | The client id. |
| alg | Must be RS256 | 

Once you have your JWT token you can submit it to Oracle as follows:
```
curl -L -X POST <Oracle Identity Cloud>/oauth2/v1/token 
    -H 'Content-Type: application/x-www-form-urlencoded' 
    -H 'Authorization: Basic <Base64 encoded client id and secret>' 
    --data-urlencode grant_type=urn:ietf:params:oauth:grant-type:jwt-bearer
    --data-urlencode assertion=<your JWT Token>
    --data-urlencode scope=<your scope>
```
See [here](https://docs.oracle.com/en/cloud/paas/identity-cloud/rest-api/op-oauth2-v1-token-post.html) for the API Documentation.

Alternatively you can use [this](https://orasites-prodapp.ocecdn.oraclecloud.com/content/published/api/v1.1/assets/CONT89581F004ECA48BC9FDF023E6BA5EBD9/native/generateJWTUserAssertion.sh?channelToken=842764f99b9a4a06a862ebc785ac9897) Oracle-supplied script.

You will need to edit the following parameters in the script:
```
APP_ID="<your client id>"
AUD="https://identity.oraclecloud.com/"
SUB="<user name>"
JTI="8c7df446-bfae-40be-be09-0ab55c655436"
KID="<certificate alias>"

ClientID="<your client id>"
ClientSecret="<your client secret>"
Scope="<your OIC scope recorded earlier>urn:opc:resource:consumer::all"
```

Either way Oracle will respond with a Bearer token. Pass this in the Authentication Header when calling your OIC flow in order to authenticate.

<img src="jwt11.png"/>


