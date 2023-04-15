---
title: Team Topologies Applied
tags: Agile
category: Agile
excerpt:
    Team size and structure is a common topic of discussion in software engineering at scale, and while the "2 Pizza" team concept popularized by Amazon can be a helpful starting point, it may not be enough. When pizzas are being delivered by the truckload, a different set of tools and strategies is required to manage the complexity and ensure successful outcomes.
---

{% img /images/TT-Logo.png 350 Team Topologies %}

Team size and structure is a common topic of discussion in software engineering at scale, and while the "2 Pizza" team concept popularized by Amazon can be a helpful starting point, it may not be enough. When pizzas are being delivered by the truckload, a different set of tools and strategies is required to manage the complexity and ensure successful outcomes.

Enter Team Topologies...

This book introduces a model for thinking about how teams are structured and interact. I see is as providing a Domain Model for thinking about how to structure teams. By aligning on the ubiquitous language provided by the model we can have better conversations about how are teams are structured.

Team Topologies provides for 4 fundamental team types and 3 modes of interaction. Before getting to these there are two concepts which underpin all of this: Conway's Law and Cognitive Load.

# Reverse Conway Manoeuver
Conway's law states that:

>Any organization that designs a system (defined broadly) will produce a design whose structure is a copy of the organization's communication structure.

I've heard this summarised as "You ship your org-chart." Whilst pithy this is not a precise characterization of the law itself. Instead, Conway's Law suggests that the way that people communicate and collaborate within an organization has a profound impact on the systems they build, and that organizations should be mindful of this when designing their structures and processes. 


![Is this the org chart or the architecture?](/images/TT-Team.png)

For a more tangible example consider an organisation that has a "Front End Team", "Services Team", "Integration Team" and "Core Banking Team". It would not be surprising to see that the Software Architecture closely models this structure. One could argue that the teams were created this way to serve the architecture - but often its the other way around - the architecture is a consequence of the organisation of the people.

Team Topologies suggests that we hack this relationship - what they call the 'Reverse Conway Manoeuver'. Design the teams to reflect the architecture you want. Change the people and interactions and the software will follow.

# Cognitive Load
Team Topologies discusses cognitive load in the context of organizing teams for optimal productivity and innovation. Cognitive load refers to the amount of mental effort required to complete a particular task or set of tasks. High cognitive load can lead to reduced productivity and increased errors, while low cognitive load can lead to boredom and disengagement. 


# Team Types
Team Topologies categorises teams as one of four types:

**Stream-aligned teams** - focused on delivering business value, work on end-to-end product delivery, and align around a specific value stream or product.

**Enabling teams** - enable and support stream-aligned teams with specific skills and tools needed to achieve their goals.

**Complicated subsystem teams** - specialize in complex or specialized subsystems, which can be shared across multiple stream-aligned teams.

**Platform teams** - build and maintain shared platforms and services that are used by multiple stream-aligned teams.

These types are also fractal in that a team may contain multiple teams. For example a Platform Team could be comprised of an Enabling Team and several Stream-Aligned Teams.

# Interaction Models

Team Topologies describes three Interaction Models for teams:

**Collaboration:** This is a model where teams work closely together, sharing skills and knowledge to achieve a common goal. Collaboration can be useful for teams that need to work together to deliver a particular feature or product.

**X-as-a-Service:** In this model, teams provide a service or capability to other teams, acting as an internal service provider. This model can be useful for teams that specialize in a particular technology or platform and can provide value to other teams.

**Facilitation:** In this model, a team acts as a facilitator, providing a framework or set of tools that enable other teams to work more effectively. This model can be useful for teams that specialize in areas such as DevOps or Agile methodologies, and can provide guidance and support to other teams.

# Team Topologies Applied
With these shiny new tools for understanding team structure in hand I started thinking about some common anti-patterns that I've seen.

## Team too big
When teams are too large the amount of communication and coordination increases. There is also an expectation that with such a large team they should be able to get lots done. This is often not the case as the communication and coordination overhead conspire to erode any potential productivity gains. 

It's also common that large teams are a consequence of combining multiple responsibilities. The resulting increase in cognitive load further impacts any expected productivity gains as not every person in the team will have the same capabilities.

One twist on this anti-pattern is a large team with a bottleneck around a specific discipline. The classic example here being the QA role. In this scenario a large pool of engineers are creating features which then pile up because the sole QA cannot test them fast enough. 

## Team too small

Small teams can work so long as you manage the Cognitive Load. A small team with high cognitive load will 

## Too much cognitive load

Regardless of team size too much cognitive load invariably results in knowledge silos in the team, burnout and frustration.

## Too little cognitive load

Under-stimulated teams typically suffer from high attrition as people flee the team before it turns into a career dead-end. For those that remain there is stagnation. The lack of challenge in the domain means that new skills are not needed and there is no drive.

## Overloaded Team Types

Once I read Team Topologies I realised that many of our challenges related to Teams that operating as more than one of the fundamental team types. For example a 'Stream-Aligned' team that managed multiple streams. Another is a team that was both 'Stream-Aligned' and a 'Platform'. It was 'Stream-aligned' in that it had a sponsor and a set of objectives but also a 'Platform' in that it was expected to provide a set of services to other teams.

In cases where a single team is operating in this manner the impact is poor prioritisation, poor encapsulation, slow page of development, blurred boundaries.

Firstly, it could lead to confusion and misalignment within the team itself. If the team is not clear on whether they are a stream-aligned team or a platform team, they may struggle to prioritize their work and allocate their resources effectively. This could result in delays, missed deadlines, and a lack of focus on the most important tasks.

Secondly, it could lead to confusion and misalignment between the team and other teams within the organization. If other teams are not clear on whether the team is a stream-aligned team or a platform team, they may not know how to interact with them effectively. This could result in misunderstandings, conflicts, and delays in delivering value to customers.

Thirdly, it could lead to duplication of effort and wasted resources. If the team is operating as both a stream-aligned team and a platform team, they may end up building duplicate functionality or services that are already provided by other teams in the organization. This could result in wasted resources and a lack of alignment with the broader organizational goals.

Furthermore, combining elements of a platform team and a complex subsystem team could also lead to duplication of effort and wasted resources. If the team is building a complex subsystem that is already provided by another team in the organization, this could result in wasted resources and a lack of alignment with the broader organizational goals.


## Mixed Interaction Models

