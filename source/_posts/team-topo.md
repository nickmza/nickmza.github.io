---
title: Team Topologies Applied
tags: Agile
category: 
- Team Topologies
excerpt: >-
  Team size and structure is a common topic of discussion in software
  engineering at scale, and while the "2 Pizza" team concept popularized by
  Amazon can be a helpful starting point, it may not be enough. When pizzas are
  being delivered by the truckload, a different set of tools and strategies is
  required to manage the complexity and ensure successful outcomes.
date: 2023-04-18 08:03:05
---


{% img /images/TT-Logo.png 350 Team Topologies %}

Team size and structure is a common topic of discussion in software engineering at scale, and while the "2 Pizza" team concept popularized by Amazon can be a helpful starting point, it may not be enough. When pizzas are being delivered by the truckload, a different set of tools and strategies is required to manage the complexity and ensure successful outcomes.

Enter Team Topologies...

This book introduces a model for thinking about how teams are structured and interact. I see it as providing a Domain Model for thinking about how to structure teams. By aligning on the ubiquitous language provided by the model we can have better conversations about how our teams are structured.

Team Topologies describes 4 fundamental team types and 3 modes of interaction. Before getting to these, there are three concepts which underpin the model: Team Size, Conway's Law and Cognitive Load.

# Team Size
The recommended team size discussed is 7-9 people, never exceeding 15. This sizing is based on Dunbar's Number which describes the number of close, trusted, relationships one may have in a group. Exceeding this number impacts team performance as trust diminishes with scale and the communications overhead increases.

# Reverse Conway Manoeuver
Conway's law states that:

>Any organization that designs a system (defined broadly) will produce a design whose structure is a copy of the organization's communication structure.

I've heard this summarised as "You ship your org-chart." Whilst pithy this is not a precise characterization of the law itself. Instead, Conway's Law suggests that the way that people communicate and collaborate within an organization has a profound impact on the systems they build, and that organizations should be mindful of this when designing their structures and processes. 

![Is this the org chart or the architecture?](/images/TT-Team.png)

For a more tangible example consider an organisation that has a "Front End Team", "Services Team", "Integration Team" and "Core Banking Team". It would not be surprising to see that the Software Architecture closely models this structure. One could argue that the teams were created this way to serve the architecture - but often it's the other way around - the architecture is a consequence of the organisation of the people.

Team Topologies suggests that we hack this relationship - what they call the 'Reverse Conway Manoeuver'. Design the teams to reflect the architecture you want. Change the people and interactions and the software will follow.

# Cognitive Load
Team Topologies discusses cognitive load in the context of organizing teams for optimal productivity and innovation. Cognitive load refers to the amount of mental effort required to complete a particular task or set of tasks. High cognitive load can lead to reduced productivity and increased errors, while low cognitive load can lead to boredom and disengagement. 

# Team Types
Team Topologies categorises teams as one of four types:

**Stream-aligned teams** - focused on delivering business value, work on end-to-end product delivery, and align around a specific value stream or product.

**Enabling teams** - enable and support stream-aligned teams with specific skills and tools needed to achieve their goals.

**Complicated subsystem teams** - specialize in complex or specialized subsystems, which can be shared across multiple stream-aligned teams.

**Platform teams** - build and maintain shared platforms and services that are used by multiple stream-aligned teams.

These team types are fractal in that a team may be composed of multiple teams. For example a Platform Team could be comprised of an Enabling Team and several Stream-Aligned Teams.

# Interaction Models

Team Topologies describes three Interaction Models for teams:

**Collaboration:** This is a model where teams work closely together, sharing skills and knowledge to achieve a common goal. Collaboration can be useful for teams that need to work together to deliver a particular feature or product. Note that this is this the highest-bandwidth (read costly) communication style.

**X-as-a-Service:** In this model, teams provide a service or capability to other teams, acting as an internal service provider. This model can be useful for teams that specialize in a particular technology or platform and can provide value to other teams.

**Facilitation:** In this model, a team acts as a facilitator, providing a framework or set of tools that enable other teams to work more effectively. This model can be useful for teams that specialize in areas such as DevOps or Agile methodologies, and can provide guidance and support to other teams.

# Team Topologies Applied
Team Topologies gives us a model and a language to discuss team structures. This allows us to highlight the presence of team 'anti-patterns', for example teams that are too big or have too much cognitive load, and suggests approaches to address them. 

In a future my intention is to discuss some of the 'anti-patterns' we've identified in our environment and describe our experiences in dealing with them. In the meantime please take a look at Team Topologies - it's an excellent read.
