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

This book introduces a model for thinking about how teams are structured and interact. I see it as providing a Domain Model for thinking about how to structure teams. By aligning on the ubiquitous language provided by the model we can have better conversations about how are teams are structured.

Team Topologies describes 4 fundamental team types and 3 modes of interaction. Before getting to these, there are three concepts which underpin the model: Ideal Team Size, Conway's Law and Cognitive Load.

# Team Size

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
With these shiny new tools for understanding team structure in hand I started thinking about some common anti-patterns that I've seen.

## Team too big
When teams are too large the amount of communication and coordination increases. There is also an expectation that with such a large team they should be able to get lots done. This is often not the case as the communication and coordination overhead conspire to erode any potential productivity gains. 

It's also common that large teams are a consequence of combining multiple responsibilities. The resulting increase in cognitive load further impacts any expected productivity gains as not every person in the team will have the same capabilities.

One twist on this anti-pattern is a large team with a bottleneck around a specific discipline. The classic example here being the QA role. In this scenario a large pool of engineers are creating features which then pile up because the sole QA cannot test them fast enough. 

## Team too small
Small teams can work so long as you manage the Cognitive Load and demand. A small team with high cognitive load will negatively impact the people. If this is combined with poor capacity management the results are predictably disastrous.

## Too much cognitive load
Regardless of team size too much cognitive load invariably results in knowledge silos in the team, burnout and frustration.

## Too little cognitive load
Under-stimulated teams typically suffer from high attrition as people flee the team before it turns into a career dead-end. For those that remain there is stagnation. The lack of challenge in the domain means that new skills are not needed and there is no drive. To be clear - this is an indictment of the organisation that created this situation, not of the people stuck in such teams.

## Overloaded Team Types
Once I read Team Topologies I realised that many of our challenges occur when we have Teams that operate as a combination of more than one of the fundamental team types. For example a 'Stream-Aligned' team that managed multiple streams. Another is a team that was both 'Stream-Aligned' and a 'Platform'. It was 'Stream-aligned' in that it had a sponsor and a set of objectives but also a 'Platform' in that it was expected to provide a set of services to other teams.

Regardless of the combination where a single team is operating in this manner the impact is poor prioritisation, poor encapsulation, slow page of development, duplication of effort and blurred boundaries.

## Mixed Interaction Models
My challenge here was that in many cases were/are relying on a single communication style - Collaboration. This results in huge amounts of communication and effort to get anything done. At a minimum we need 3 teams to get anything of substance done. When they are all relying on Collaboration it is an expensive and suprisingly fragile process. Another consequence of this is that teams bleed implementation detail to the extent that teams know a lot of detail about what each is doing.

# Where to from here?

If any of this sounds familiar you may be wondering what can be done.

Firstly, recognise which of the team types are in play. Then agree on the interaction model for interactions between them.

At this point you have a template for where you want to be. Now you need to think about how you can get there. The key is to make small changes - testing as you go to ensure that you are getting the right results.

Start by looking for the antipatterns above and use this to guide your decisions. Ask yourself:
-  Which teams are too large and could be split? 
-  Could multiple small teams be merged? 
-  Would moving responsibilities between teams help balance cognitive load?


