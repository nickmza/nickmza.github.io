---
title: Team Topologies Applied
tags:
---

A frequent point of discussion in software engineering at scale is that of team size and structure. Whilst Amazon's much-vaunted '2 Pizza' teams form the starting point for most companies once you reach the point where pizza's are being delivered by the truck-load you will need a new set of tools to deal with the complexity.

Enter Team Topologies...

# Reverse Conway Manoevre
Conways law states that

>Any organization that designs a system (defined broadly) will produce a design whose structure is a copy of the organization's communication structure.

In other words "You ship your org-chart." For a more tangible example consider an organisation that has a "Front End Team", "Services Team", "Integration Team" and "Core Banking Team". It would not be surprising to see that the Software Architecture closely models this structure. One could argue that the teams were created this way to serve the architecture - but often its the other way around - the architecture is the a consequence of the organisation of the people.

What TT suggests is that we hack this relationship - what they call the 'Reverse Conway Manoeuver'. Design the teams to reflect the architecture you want. Change the people and the software will follow.


# Team Types
**Stream-aligned teams** - focused on delivering business value, work on end-to-end product delivery, and align around a specific value stream or product.

**Enabling teams** - enable and support stream-aligned teams with specific skills and tools needed to achieve their goals.

**Complicated subsystem teams** - specialize in complex or specialized subsystems, which can be shared across multiple stream-aligned teams.

**Platform teams** - build and maintain shared platforms and services that are used by multiple stream-aligned teams.

# Interaction Models
**Collaboration:** This is a model where teams work closely together, sharing skills and knowledge to achieve a common goal. Collaboration can be useful for teams that need to work together to deliver a particular feature or product.

**X-as-a-Service:** In this model, teams provide a service or capability to other teams, acting as an internal service provider. This model can be useful for teams that specialize in a particular technology or platform and can provide value to other teams.

**Facilitation:** In this model, a team acts as a facilitator, providing a framework or set of tools that enable other teams to work more effectively. This model can be useful for teams that specialize in areas such as DevOps or Agile methodologies, and can provide guidance and support to other teams.
