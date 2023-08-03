---
title: Sidecar Logging in Openshift with Fluentd and Elasticsearch
tags: devops
excerpt:
  How to set-up a 'sidecar' container to log events from your Kubernetes applicationt to Elasticsearch using Fluentd.
---

We use Elasticsearch for logging in our Openshift environments. This means that each application needs to figure out how to aggregate their logs and ship them to Elasticsearch. This duplication of effort is wasteful and increases maintenance costs as a change at Elasticsearch level must then be absorbed by each application. This approach also amplifies the impact of a security vulnerability such as the recent [Log4j](https://nvd.nist.gov/vuln/detail/CVE-2021-44228) vulnerability in that it requires patching in multiple places. One solution would be to have a single 'certified' logging library however we would need one for each technology stack we use. There is another way however - the sidecar deployment.

Kubernetes (k8s) allows for multiple containers to run in the same pod. These containers can share resources and dependencies, communicate with one another, and coordinate when and how they are terminated. In the 'Sidecar Deployment' we have one container providing a service for another. In our case what we are looking for is that the main application writes it's logs to file and the sidecar then manages the process of shipping the logs to Elasticsearch.

The solution will look like this:
<iframe frameborder="0" style="width:100%;height:400px;" src="https://viewer.diagrams.net/?highlight=0000ff&nav=1#R1Zddk5owFIZ%2FjZftQPADL%2F3cbWtbZ%2BzMuledLBwhbSRMCAr765tIICCu63bs1L0y581JJO85Dx8dZ7LN7jiOw6%2FMB9pBlp91nGkHIRt1kfxRSl4oA3dYCAEnvk4ywoo8gxYtrabEh6SRKBijgsRN0WNRBJ5oaJhztm%2BmbRht%2FmuMA2gJKw%2FTtvpAfBEWqtuzjH4PJAjLf7YtPbPFZbIWkhD7bF%2BTnFnHmXDGRDHaZhOgyrzSl2Ld%2FIXZ6sI4ROKSBaOpn%2FX5eL1Zfrv%2F%2BeM5%2FzRnnz%2BUNu8wTfWJv7grKSyZry9b5KUXnKWRD2o7u%2BOM9yERsIqxp2b3svpSC8WW6ukdcEGkjyNKgkhqgqmE9kXrc6h0yGqSPsQdsC0InsuUcrZ0WHeU3dfx3tQHuVoLa7VBZdGw7omg2tvYJgfauTe46LasAl92kQ4ZFyELWITpzKhjY6YlI5OzYMqog4W%2FQIhcI4FTwZoGQ0bEujZ%2BVFt97OlomumdD0FeBpE87roe1Fap0Cw7RI11S%2BBE%2BgVciy%2FWMmEp9%2BBc12mQMQ9AnMkbFHnKzLOdwYFiQXZNZK9e5bJTDSoLFgQkCqQ4YZHAJJLeHHfCK5jgJC5uWxuSqW64BiGo3yQEWW1CKorqhFTi1b3r%2Fk9CDBWPtZmbJgRdSIhzU4SgFiGjOL41Omx0c3T0383zQwbHTW6IMZBUnJ0h5u%2FhcC6Ew74pOJxTjw8pzIms53H95ctirIZeToksM3def%2BF6Khpi8VQJ2PsdHNrkeyrkNqD1pOgIu3elp43d5Mmx2zz1T%2BDk%2FiuaBi2jZxQn8mU0Acy98D173UWDo3tXr%2BX18ITXw7d7LUPzbXKYq33hObM%2F"></iframe>

This approach means that we can setup our logging container once and then use it with all of our application containers, regardless of the technology stack. In addition we can inject the sidecar via our Policy Framework when a pod is deployed. So even if the team forget to enable logging it will be there automatically. Lastly this provides isolation so that if the logging process fails the main application can continue working.

# Logging Container
I'm going to use Fluentd to process the logs and send them to Elasticsearch. To do this I need to add the Elasticsearch plugin to the Fluentd Docker image and add my config file.

```
FROM fluent/fluentd:edge-debian
USER root
RUN ["gem", "install", "fluent-plugin-elasticsearch", "--no-document", "--version", "5.3.0"]

COPY fluent.conf /fluentd/etc/

COPY entrypoint.sh /bin/
RUN chmod +x /bin/entrypoint.sh

USER fluent
```

# Fluentd Configuration

I'm starting with a minimal configuration. We tail the log at 'var/log/mcb.log' and tag the lines with 'mcb'. Fluent will then match any log tagged with 'mcb' and send it to Elasticsearch.

I ran into some challenges here as I needed a specific index name to be used. Whilst I set the index_name property this did not seem to have any effect. The culprit was the logstash_format property. If this is set to 'true' then index_name is ignored. Switching to 'false' fixed the problem. 

> If you do run into problems set the @log_level to debug at plugin level. In addition setting  with_transport_log to true will allow you to see the detailed trace associated with how the data is sent to Elastic.

```
<source>
  @type tail
  path /var/log/mcb.log
  pos_file /var/log/td-agent/mcb.log.pos
  tag mcb
  <parse>
    @type none
  </parse>
</source>

<match mcb>
  @type elasticsearch
  @log_level debug
  host es-dev.mcb.local
  port 9200
  user elastic
  password elastic
  scheme https
  ssl_verify false
  logstash_format false
  index_name pfmdevlog
  with_transporter_log true
</match>

```
# Configuring the sidecar
We start by creating a volume for the containers to share data. This can be added to your K8S Deployment as follows:

```
  volumes:
    - name: applog
      emptyDir: {}

```
An [emptyDir](https://kubernetes.io/docs/concepts/storage/volumes/) volume is first created when a Pod is assigned to a node, and exists as long as that Pod is running on that node. 

Next we add the sidecar container to the deployment:
```
  containers:
    - name: app-container
    ...
    - name: mcb-logger
      image: >-
        docker.digitalfactory.mcb.local/mcb-sre/mcb-logger@sha256:cfd8bdd41f4fa5fbe031856444a1d272a61e7f70d6f1e42d435a61c8f856ba6d

```
Finally we add a volumeMount to both containers:

```
      volumeMounts:
        - name: applog
          mountPath: /var/log/
```

In summary we now have 2 containers running in the pod who have a shared directory located at '/var/log'.

# Use of Environment Variables
To this point I've hardcoded the Fluentd settings. This will not work for everyone as we use different indexes for the various applications. I need to allow each application to inject these settings when the pod is started. Fortunately Fluentd allows you to execute arbitrary Ruby code when the configuration file is parsed. This allows us the opportunity to replace values from the local ennvironment. Here's the updated configuration:

```
<source>
  @type tail
  path "#{ENV['LOG_FILE'] || '/var/log/mcb.log'}"
  pos_file /var/log/td-agent/mcb.log.pos
  tag apache.access
  <parse>
    @type none
  </parse>
</source>

#<match **>
#  @type stdout
#</match>

<match apache.access>
  @type elasticsearch
  @log_level "#{ENV['LOG_LEVEL'] || 'debug'}"
  host "#{ENV['ES_HOST'] || 'es-dev.mcb.local'}"
  port "#{ENV['ES_PORT'] || 9200}"
  user "#{ENV['ES_USER'] || 'elastic'}"
  password "#{ENV['ES_PASSWORD'] || 'elastic'}"
  scheme https
  ssl_verify false
  logstash_format false
  index_name pfmdevlog
  with_transporter_log true
</match>

```

# Testing
Once the pod is started we can now see both containers running:

<img src="sl1.png"/>

If we then jump to the application container we can test by manually adding a log entry:

<img src="sl2.png">

The Fluentd running in the logging container takes this log line and sends it to Elasticsearch.

<img src="sl3.png">

# Next steps
At present the Fluentd configuration is minimal so I'll need to provide more flexibility. The simplest method would be to use a ConfigMap to replace the Fluentd configuration entirely. There may be other options - let me know in the comments if you have a suggestion. I also need to get up our policy framework to automatically inject the sidecar if it's not part of the applications configuration.