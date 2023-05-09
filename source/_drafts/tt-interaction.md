---
title: From Interrupted to Empowered - The Benefits of Choosing the Right Communication Mode
tags: Agile
category: 
- Team Topologies
excerpt: >-
  This post looks at what happens when teams pick an interaction model that does not suite the task at hand.
---

In a previous [blog](https://nickmck.net/2023/04/18/team-topo/) I discussed the four fundamental team types and 3 interaction patterns described in Team Topologies.  One of the things I love about this book is that it provides a consistent way to think about teams, their purpose and how they interact. The simplicity of the model makes it easy to spot potential issues in team structures and offers some potential solutions.

With this in mind I though I’d share some of what we’ve learned on our journey at MCB over the last couple of years. There’s too much for a single post so I’ll break this up into a series of posts, each focussing on a specific challenge. First up: Inappropriate Communication Patterns.

In this scenario the communication model being used is inappropriate for the task at hand.  In our case, we were using Collaboration when X-as-a-Service (XAAS) or Enabling would have been more appropriate. This is a problem because Collaboration is the most expensive form of communication. When mis-applied it results in blocking delays because teams are waiting for each other, or we are requiring teams to be in the same space in order to collaborate. **Is this strong enough?

# XAAS vs Collaboration
In one case we had teams needing access to log information from another team. For security purposes the requesting teams were not allowed access to the logs directly. The teams would typically address this requirement via Collaboration. How this worked in practice is that the requesting team would contact the team with the logs, ask for the specific records and then wait for the information to be extracted and returned. This would result in the requesting team waiting and the fulfilling team being interrupted continually with requests for logs. 

Once this pattern was recognised the solution was simple. The team created a self-service tool that allowed the requesting team to access the log data without compromising security. What was once a Collaboration interaction is now a XAAS interaction. Then teams are not blocking or interrupting each other.

> Return on Investment. In this case the team invested time to develop a tool that would help them change the interaction pattern.  Development, and maintaining the result can be expensive though so it’s important to quantify the effort in the context of the potential gains. The way I typically look at it is this - how much time are we currently spending doing X? How much time will it take to create a tool to do this and then maintain it? With these 2 data points we can quickly determine how long it will take to get a Return on Investment. If the ROI is measured in years there's probably something else we should rather be doing. 

# Enablement vs Collaboration
In another case one of our Platform teams was using Collaboration with multiple other teams to support their use of the platform. The issue was that the Platform team was, in reality, owning each other team’s implementation on the platform. Consequently when a change needed to be made, or there was an issue with the implementation Collaboration between the 2 teams was then required. 

The solution was to change the interaction model from Collaboration to Enablement. Teams needed to take ownership of their implementations. The platform team would then Enable them with tooling, training and best practices. The result is that the Platform team has more time to focus on improving the platform and the other teams are no longer blocked waiting for the Platform team to address their requirements. 

# The results
We have successfully moved X and Y blah blah 

It’s early days but these changes are showing results. Interruptions and unplanned work have been reduced. Teams are taking ownership of their own implementations. 

> These are specific examples and solutions from our environment. Remember that your mileage may vary and please be aware of the dangers of installing someone else’s solution. Move slowly and measure as you go. 

As advice I would say to look at teams and how they are interacting through the lens of the 4 team types and 3 interaction models. Pay specific attention to where you are using Collaboration and ask if this could not be simplified.

> Keep in mind that interaction models between teams change over time. What often starts as a Collaborative engagement may morph to XAAS or Enablement. It's important to note that circumstances may require a temporary or permanent shift in this relationship. Also - there may be multiple interaction models between teams based on context*


* although this could also point to a potential overloading of the team. A topic for a future post.