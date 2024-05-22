---
title: Deploying Backstage on Openshift in a Corporate environment.
tags: devops
category:
  - Backstage
ogimage: bso-1.png
excerpt: Some tips on deploying Backstage on Openshift.
date: 2024-05-22 05:42:06
---

<img src="bso-1.png"/>

We are currently evaluating [Backstage](https://backstage.io/) as a developer platform. I'll be doing a separate article on what Backstage actually is and our experiences with it a little later. For now I'm assuming that if you are reading this you already know and are wrestling with deploying it into your environment. Here are some of the things that took me some time to figure out...


# Container Security Context
The Dockerfile that ships with Backstage uses a least-privilege account (called 'node') to run the application. Once we had deployed the container to Openshift we started receiving an error message 'cannot find module '/app/packages/backend' on start-up. The issue was that we needed to set the container's security context as follows:

```
spec:
  securityContext:
    runAsUser: 1000
```
The 1000 here tells Openshift to run as the first non-system user. In this case the 'node' account used in the Dockerfile.

# Self-signed certificates
Internally all certificates are self-signed which causes HTTPS connections to fail unless properly configured. It wasn't immediately apparent how to achieve this in Backstage. The answer is the NODE_EXTRA_CA_CERTS environment variable. Just set this to a path to your internal certificate. In my case I have multiple certificates to deal with so I concatenated the certificates into a single file.

# Proxy
Internet access from our environment is via a proxy server. In addition, some plugins required us to bypass the proxy. The first step is to enable the proxy globally in your 'packages/backend/srv/index.ts':

```JAVASCRIPT
const proxyEnv =
  process.env.HTTP_PROXY ||
  process.env.HTTPS_PROXY;

if (proxyEnv) {
  console.log(`Setting Proxy:${proxyEnv}.`);
  const proxyUrl = new URL(proxyEnv);

  const token = proxyUrl.username && proxyUrl.password ? 
		`Basic ${Buffer.from(`${proxyUrl.username}:${proxyUrl.password}`).toString('base64')}` : undefined;

  setGlobalDispatcher(
    new ProxyAgent({
      uri: proxyUrl.protocol + proxyUrl.host,
      token: token,
    }),
  );
}else{
  console.log("No proxy set.");
}
```
This will then use a proxy for all calls as long as the HTTP_PROXY or HTTPS_PROXY environment variables are set. 

To bypass the proxy for specific addresses simply set the GLOBAL_AGENT_NO_PROXY environment variable with the hosts that do not require the proxy.

> Be sure to add localhost to you proxy exclusion list! Techdocs and Search will not function otherwise. 

# Conclusion
Diagnosing and fixing these issues - especially when you need to rebuild and upload container images is time consuming. I hope this list can save someone a little time down the road. I'll update this list if I uncover an more OpenShift related issues.

