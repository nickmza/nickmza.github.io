---
title: From Interrupted to Empowered - The Benefits of Choosing the Right Communication Mode
tags: Agile
category: 
- Team Topologies
excerpt: >-
  This post looks at what happens when teams pick an interaction model that does not suite the task at hand.
---

In a previous [blog](https://nickmck.net/2023/04/18/team-topo/) I discussed the four fundamental team types and 3 interaction patterns described in Team Topologies.  One of the things I love about this book is that it provides a consistent way to think about teams, their purpose and how they interact. The simplicity of the model makes it easy to spot potential issues in team structures and offers some potential solutions.

With this in mind I though I’d share some of what I've over the last couple of years.  There’s too much for a single post so I’ll break this up into a series of posts, each focussing on a specific challenge.  First up: Inappropriate Communication Patterns.

In this scenario the communication model being used is inappropriate for the task at hand.  The most common form of this I've seen is using Collaboration when X-as-a-Service (XAAS) or Enabling would have been more appropriate. This is a problem because Collaboration is expensive. When mis-applied it results in blocking delays because teams are waiting for each other, or we are requiring teams to be in the same space in order to collaborate. **Is this strong enough? No needs rewording.

# XAAS vs Collaboration
Something I've found helpful over the years is looking for opportunities to turn Collaboration ino XAAS. These can be easy to spot: look for cases where you have one team requesting something from another. This could be a request for some piece of information ("Please send us logs for X") or an action to be performed ("We need a backup of this database."). When handled via Collaboration, especially if done in an ad-hoc fashion, the team on the receiving end experiences this request as an interruption. This can cause context-switching which delays other, possibly more valuable, work for the receiving team. This also ensures a delay for the requesting team as they need to wait for the results which causes more context-switching.

Once this pattern of interaction is identified you need to see if it can be turned into XAAS. For example one of our teams at MCB created a self-service tool that allows teams to access sensitive log data without compromising security. In other cases teams have created Bots to automate requesting VM's and other infrastructure to be provisioned. What was previously a high-bandwidth, blocking activity is now an asynchronous, self-service one.

> Return on Investment. In some cases teams will need to invest time to develop a tooling to enable the change in interaction pattern.  Development, and maintaining the result can be expensive though so it’s important to quantify the effort in the context of the potential gains. The way I typically look at it is this - how much time are we currently spending doing X? How much time will it take to create a tool to do this and then maintain it? With these 2 data points we can quickly determine how long it will take to get a Return on Investment. This is very useful when positioning these activities with Product Owners as they can clearly see the benefits of the investment. 

# Enablement vs Collaboration
Another common scenario is where one team is dependent on another due to access to a specific application and/or skill-set. The result is that one team cannot operate independently of the other. In cases where XAAS is not appropriate moving to Enablement can provide benefits.

In this approach one team coaches the other to build the capabilities to make the other independent. This would typically require training, documentation, tooling and QA activities. If successful you will have managed to largely remove a dependency between teams which will allow each to be more efficient.

I would recommend that you take an iterative approach to this rather than trying to make a team totally independent in one go. Identify the simple use-cases first and then hand these over. Once the team has built competency and confidence with the simple scenarios you can gradually add complexity.


# The results

As advice I would say to look at teams and how they are interacting through the lens of the 4 team types and 3 interaction models. Pay specific attention to where you are using Collaboration and ask if this could not be simplified.

> Keep in mind that interaction models between teams change over time. What often starts as a Collaborative engagement may morph to XAAS or Enablement. It's important to note that circumstances may require a temporary or permanent shift in this relationship. For example, creating a new service from scratch may require that the teams switch to Collaboration for a short period to agree the contract. 
