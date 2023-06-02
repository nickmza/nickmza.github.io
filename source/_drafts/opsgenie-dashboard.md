---
title: Real-time Dashboards with OpsGenie and PowerBI
tags:
- devops
excerpt:
    We needed to visualise alerts from OpsGenie in real-time. Here's how we did it using PowerBI and Power Automate.
---

<img src="og7.jpeg"/>

We use OpsGenie for Alert Management. Our intention is to have all alerts surfaced in OpsGenie as this gives us an end-to-end view of what's happening in the environment. Whilst OpsGenie provides a serviceable user interface and mobile tools one thing it lacks is the ability to build dashboards to visualise the information. This week I built a solution using PowerBI.

There are many ways to do this but I did not want to build something from scratch. Ideally I wanted something that would work with minimal or no code and allow us to easily create new dashboards. Since PowerBI is my go-to for reports and dashboards I wanted to see if there was a way to get it to talk to OpsGenie. Turns out there is, with a little help from PowerAutomate.

Here's the plan:
<iframe frameborder="0" style="width:100%;height:200px;" src="https://viewer.diagrams.net/?highlight=0000ff&nav=1#R1VZNc5swEP01HNsBZLB7rD%2BaZMadZOpDm6MMG1AiECOEgf76LmYxpjROOuNJnJO1Tytp9%2B3bNRZbJNWV5ln8XYUgLdcOK4stLdd1mO3jT4PULTL1Ji0QaRGSUw9sxG8g0Ca0ECHkA0ejlDQiG4KBSlMIzADjWqty6Pag5PDVjEcwAjYBl2P0pwhN3KIzz%2B7xaxBR3L3s2LST8M6ZgDzmoSqPILay2EIrZdpVUi1ANuR1vLTnvj2zewhMQ2pec2AVBPdCOU%2Fxjy%2BpcB4XVTBdf6Ji5KbuEoYQ8ydTaROrSKVcrnp0rlWRhtDcaqPV%2B6yVyhB0EHwEY2oqJi%2BMQig2iaRdDFjXv%2Bj83rhvjM9eZy6r481lTVYbaxPgsxQQlKtCB3Ai705KXEdgTvixQ6FQ4aASwHjwnAbJjdgN4%2BAktejg11cDF1SQ%2FygO3bvjsqCXbrP8ClIBo6qVsTCwyfg%2B6RJbccg4z7O2OR5E1VSOqNyBNlCdJnOcPB2Yka6psVnXseVRmxAUH3VIh52dLf8jSXlv3YEWmDvo8%2BubvVLf3nvqm430fadKZMO1vyLROEAvQufu5MKE7o1o2xgNPBFpZLm%2BxOfnWyTRj5rVkhueowT%2BJhL%2FjbJmGdRSYAtohr4vkLttm2W9PQA8eIr2LXRbGLwGCM%2FbbnG88xRgYr9cgOlbFmD2rpPGuaxJM%2F0Ik2b670kzv7mEAeP5bzZg0Ow%2FOPd7R5%2FtbPUH"></iframe>

# Streaming Datasets

You could create this solution by pushing events from OpsGenie to a database but I wanted something simple and real-time. As it happens, PowerBI provides a facility for this in the form of 'Streaming Datasets'. Essentially this provides an API for you to push events to and then build reports on top of this store. Events can be real-time only or you can retain the history.

I started by creating a Streaming Dataset with the following fields:

<img src="og1.png" width="300px"/>

> You can create your Streaming Dataset from your PowerBI workspace by selecting New...Streaming Dataset and follow the prompts.

Once the Streaming Dataset is created you can post your data my making an API call. For example:

{% codeblock lang:bash %}

curl --include \
--request POST \
--header "Content-Type: application/json" \
--data-binary "[
{
\"Integration\" :\"AAAAA555555\",
\"Created\" :\"2023-05-26T11:24:08.057Z\",
\"Severity\" :\"AAAAA555555\",
\"Message\" :\"AAAAA555555\"
}
]" \
"https://api.powerbi.com/beta/XXX/datasets/XXX/rows?experience=power-bi&key=XXX"

{% endcodeblock %}

# Power Automate
The next step was to get OpsGenie to update the Streaming Dataset when a new alert was created. Whilst OpsGenie does provide an HTTP Integration it does not allow you to control the request format. I needed something that could format the request and possibly add some additional logic. Not wanting to build a custom service for this I used Power Automate.

This turned out to be quite a simple flow to build. It's an HTTP trigger which in turn calls the built-in PowerBI connector.

<img src="og2.png"/>

# OpsGenie

Last thing to do was to get OpsGenie to call the Power Automate flow when a new alert is created. This can be done by using the OpsGenie Webhook integration.

<img src="og3.png"/>

# Testing

At this point we have an OpsGenie Webhook integration that will call the Power Automate flow when a new alert is created. This will, in turn, call the PowerBI Streaming Dataset passing a subset of the alert information. Let's give it a test...

First thing to do - create a Test Alert.

<img src="og4.png"/>

Next check if the alert was received in Power Automate.

<img src="og5.png"/>

# Power BI Report

At this point the data is waiting in the Streaming Dataset. All that remains is to create a Power BI Report or Dashboard on top of it.

Here's a simple one I created showing Alerts over time as well as the details and split by team.

<img src="og6.png"/>

# Next Steps

At present I get a real-time feed as new alerts are created but I would also want to remove alerts once they are closed on OpsGenie. This would allow me to create a Red-Amber-Green view of the various systems. I think this can be done by adding some additional logic to the Power Automate flow and creating a new Streaming Dataset to store the overall status. Will try this next. 

