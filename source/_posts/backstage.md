---
title: Building Developer Portals with Backstage
tags: devops
category:
  - Backstage
ogimage: bs.png
excerpt: >-
  We've just completed a proof of concept on Backstage - an Internal Developer
  Portal from Spotify. Here's what we learned...
date: 2024-06-19 16:11:22
---


We've just completed a proof of concept on Backstage. Here's what we learned...

Backstage is an Internal Developer Portal (IDP) originally created at Spotify and then open-sourced to the Cloud Native Computing Foundation. Out of the box it provides a Software Catalog, Developer Documentation and Software Templates. There is also a thriving plugin ecosystem that expands Backstage's capabilities even further.

# What problem are we solving here?

We didn't initially go out looking for an IDP, rather we found ourselves looking to address a number of specific issues.

In our last DevEx survey, the ability to find information quickly was raised as an issue. The feedback indicated that whilst the quality of information was generally good it often took time to find the right information. We want people to get to the relevant documentation quickly and have it in a consistent format.

Another issue is the ability to provide a standardized diagnostic and operational interface to our applications. Presently, depending on the team and platform, there are differences in how you access logs, view stats and traces and initiate operational procedures. This means that there is a degree of domain knowledge required for each application. This domain knowledge makes it difficult for our support team to work independently.

Lastly we want to make it easier for people to discover available services and their owners. In a recent Incident it took too long to solve a problem because the people on the call had an imperfect understanding of how the application fit together. As a result we completely missed a failing component. We want to make it easier for people to understand how the various parts of our systems relate to one another.

We started looking at how other organisations are addressing these issues and discovered Backstage. 

# The Proof of Concept
In order to effectively evaluate Backstage you need to populate it with your own data and configure plugins relevant to your environment. We gave some of the teams a little training on the Backstage Catalog Descriptor format and asked them to model some of their applications.

We then set about configuring Backstage and some plugins... 

## Software Catalog
The Software Catalog lies at the heart of Backstage. It allows you to visualise how your applications fit together and who owns what parts. For example you can start with your Mobile or Web Application, and then find all the services it depends on and who owns them. You can then drill through to see the next level of services and infrastructure. 

There is a fully-featured search facility so that you can find items by Type, Tag, Owner etc. 

<img src='bs7.png'/>

Or you can navigate through the information as a dependency graph.

<img src='bs2.png'/>


## Kubernetes
As you navigate through the Software Catalog you can view Kubernetes information and status of each service. This has proved to be invaluable for us as we run across multiple namespaces and security is configured to minimise access. This complicates the diagnostic process as we would typically have to log into multiple namespaces to check logs and service status. With this plugin we can access logs and status immediately.

You get an overview of the deployment...

<img src='bs3.png'/>

As well as detailed information and logs per Pod.

<img src='bs4.png'/>

## Azure DevOps
We can see the status of our Build Pipelines and latest Pull Requests in the context of each component. This makes it easy to spot if there have been recent changes to a component or if we have had a recent build failure.

<img src='bs5.png'/>

## SonarQube
The SonarQube plugin allows us to see a summary of the metrics for each component: Smells, Code Coverage, Security etc.

<img src='bs1.png'/>

## Prometheus
Where we have Prometheus Traces and Alerts configured these are displayed in the context of the component.


<img src='bs8.png'/>

<img src='bs9.png'/>

## OpsGenie
Any OpsGenie alerts or incidents related to the component can be seen in context. In addition the Analytics and On-Call information for all the teams is available in a separate view.

<img src='bs6.png'/>

# Initial Impressions

## Single Pane of Glass

Backstage centralises information that would typically be spread across multiple sites. Our teams have information stored in Confluence, OpenShift, Grafana, Prometheus, OpsGenie and Sonar. Backstage surfaces this information on a single 'pane of glass' - I can rapidly jump into the system that I am looking for. Yes you could do this using a page in Confluence but having the high-level information rendered in Backstage saves me time and means I don't have to click through to check if everything is OK. This is nice - but it's not life-changing. 

Where this facility really starts to shine is where I move out of a team I'm familiar. I know where all *my* links are but teams do things slightly differently. This means that you need to know where to look for a team's information. Backstage solves this by providing me access to each team's information in a consistent manner and independent of my specific access rights. For example - I would not typically have access to each team's Openshift namespace. Backstage allows me to quickly check if a service has failing Pods and view basic log information.

## Different Perspectives
Backstage allows teams to model their own software in terms of how it is structured, the API's they provide and their dependencies on other team's services. This has created a view our environment that would have been difficult to create previously. With Backstage I can quickly navigate through layers of services and infrastructure. At any point I can then drill into development information (source code, PR's, Sonar) or into the operational information (K8S, Prometheus) for each service. This makes it easy for teams to discover services but it also promotes discussion about how best to model our environment. The use case that the teams immediately latched upon was that this simplifies impact analysis. With a few clicks you can see all the teams that would be affected by a potential change.

## Maintenance
One of the things I really like about Backstage is that it does not require us to create new data via it's UI. Rather a yaml file is added to the repo and this is automatically picked up. This has been a sticking point for us in the past - where information is stored separately it means people need to remember (or more often be mercilessly nagged) to update these systems. With this approach the yaml is maintained as part of the application's source code. 

## Challenges
Whilst Backstage makes it easy to import your information you still need to ensure that team's create their catalog entries in the first place and that these entries reflect reality.  Since the information in Backstage links to other systems it is easy to spot missing or broken links but getting the modelling right could be a significant challenge. One specific concern I have is in the level of granularity. Modelling at too coarse a level means you would miss important details. Modelling at too fine a level would create potentially hundreds of artifacts and obscure important information. 

Lastly, Backstage is not a packaged application when running on-prem. There is a steep learning curve in terms of how plugins are added and configured. If you are not familiar with modern web development this could be a significant challenge. Fortunately, there is a cloud-based version of Backstage available which could be a better option for some. Documentation is generally good but we have had some challenges which I have written about [here](https://nickmck.net/2024/05/22/backstage-openshift/).

## Worth the effort?
A question we wanted to answer is "Is the effort to describe your application and services worth it compared to the potential value that Backstage brings?". The potential benefits for us include improved discoverability of services, access to documentation, teams being more independent and faster resolution of incidents. The cost is that we need teams to describe their application. Here's what a typical Backstage entry looks like:

``` YAML
---
# https://backstage.io/docs/features/software-catalog/descriptor-format#kind-system
apiVersion: backstage.io/v1alpha1
kind: System
metadata:
  name: mcb-system
spec:
  owner: mcb-engineering
---
# https://backstage.io/docs/features/software-catalog/descriptor-format#kind-component
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: mcb-component
  annotations:
    backstage.io/source-location: url:https://dev.azure.com/****/
    sonarqube.org/project-key: "******"
    dev.azure.com/project-repo: "******"
    dev.azure.com/host-org: dev.azure.com/*****
    opsgenie.com/component-selector: "responders:'***'"
    prometheus.io/rule: jvm_memory_used_bytes, jvm_threads_live_threads
    prometheus.io/alert: "***-availability"
    backstage.io/techdocs-ref: url:https://***
    backstage.io/kubernetes-label-selector: 'app=***bpm-application***'
spec:
  type: service
  lifecycle: production
  owner: mcb-engineering
  system: mcb-system
  providesApis: [mcb-api]
---
# https://backstage.io/docs/features/software-catalog/descriptor-format#kind-api
apiVersion: backstage.io/v1alpha1
kind: API
metadata:
  name: mcb-api
spec:
  type: openapi
  lifecycle: production
  owner: mcb-engineering
  system: mcb-system
  definition: 
    $text: https://***
---

```

The feedback so far to this question is 'Yes'. That said we are proceeding slowly adding services one at a time and tweaking Backstage as we go. Ultimately though we will measure the success of Backstage by whether or not people use it.  

# Next up
There are 2 other aspects of Backstage that I've not discussed here: Templates and Documentation. We've had a look at these facilities but I'll cover them in a future post.












