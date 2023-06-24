---
title: Release the KrakenD. Part 1 - Metrics
tags:
  - devops
category:
  - Release The KrakenD
ogimage: k1.png
excerpt: >-
  We are currently looking for a API Gateway. Ideally I'd like something that is
  Open Source, DevOps friendly and has first class support for OpenApi in
  addition to its general API Gateway features. Today I'm playing with the
  excellently named KrakenD...
date: 2023-06-23 12:39:31
---


<img src="./k1.png" />

We are currently looking for a API Gateway. Ideally I'd like something that is Open Source, DevOps friendly and has first class support for OpenApi in addition to its general API Gateway features. Today I'm playing with the excellently named [KrakenD](https://www.krakend.io/)...

# The Scenario
To put KrakenD through its paces I've got a simple API created in Node that uses JWT for authentication. I'm going to use KrakenD to see how I can add Logging, Metrics, Security and Request/Response modification. This will most likely be too much for one post so I'll split these into a series.

# Running KrakenD locally
I'm running KrakenD on my local machine via Docker. Here's the command:
```
docker run -p 8080:8080 -v $PWD:/etc/krakend/ devopsfaith/krakend run --config /etc/krakend/krakend.json
```

Before doing this though you'll need to create a krakend.json file.

# Basic Setup
KrakenD uses a single json file for its configuration. You can create this manually or via the [designer](https://designer.krakend.io/#!/).

Here's my initial setup:

```
{
  "$schema": "https://www.krakend.io/schema/v3.json",
  "version": 3,
  "name": "KrakenD - API Gateway",
  "extra_config": {
  },
  "timeout": "3000ms",
  "cache_ttl": "300s",
  "output_encoding": "json",
  "endpoints": [
    {
      "endpoint": "/v1/atms",
      "method": "GET",
      "output_encoding": "json",
      "backend": [
        {
          "url_pattern": "/bank/atms/",
          "encoding": "json",
          "sd": "static",
          "method": "GET",
          "is_collection": true,
          "host": [
            "https://foo.azurewebsites.net"
          ],
          "disable_host_sanitize": false
        }
      ]
    }
  ]
}
```

> Note the 'is_collection' flag - this is required when the service returns an array. If not present you would receive an error 'json: cannot unmarshall array into Go value of type map[string]interface {}'.

This is setting up KrakenD to act as a gateway for my Bank Api running in Azure. To test this I hit 'http://localhost/v1/atms' and I should see the result from 'https://foo.azurewebsites.net/bank/atms'.

<img src="k2.png"/>


# Metrics
To configure KrakenD to collect metrics you add the following to the extra_config section.

```
  "extra_config": {
    "telemetry/metrics": {
      "collection_time": "60s",
      "proxy_disabled": false,
      "router_disabled": false,
      "backend_disabled": false,
      "endpoint_disabled": false,
      "listen_address": ":8090"
    }
  },
  ```

  The metrics can then be accessed via the port specified by the listen_address parameter. Hitting http://localhost:8090/__stats results in:

  ```
  {
"cmdline": [
"krakend",
"run",
"--config",
"/etc/krakend/krakend.json"
],
"krakend.proxy.latency.layer.backend.name./bank/atms/.complete.false.error.false.50-percentile": 0,
"krakend.proxy.latency.layer.backend.name./bank/atms/.complete.false.error.false.75-percentile": 0,
"krakend.proxy.latency.layer.backend.name./bank/atms/.complete.false.error.false.95-percentile": 0,
  ```

You can consume these via the endpoint but KrakenD ships out of the box with [Opencensus](https://opencensus.io/) support.

<img src="k3.png" width="500px"/>

To configure Zipkin for example just add the following to extra_config:

```
    "telemetry/opencensus": {
      "sample_rate": 100,
      "reporting_period": 0,
      "exporters": {
        "zipkin": {
          "collector_url": "http://192.168.68.113:9411/api/v2/spans",
          "service_name": "krakend"
        }
      }
    }
```

You can then view your traces in Zipkin:
<img src="k4.png">

# Next Steps
So far KrakenD has been a well-behaved sea monster. Set-up was easy and the documentation is clear. The Designer is really cool as it helps you create a mental model of how the json config is structured but also helps you see what features are available without having to read all the docs. At this point I have a simple example working and logging and metrics configured so that I can see what's happening. In the next article I'm going to play with JWT Token validation and see if I can get the wee beastie to validate our tokens for us.
