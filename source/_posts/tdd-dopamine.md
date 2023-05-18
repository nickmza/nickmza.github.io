---
title: TDD - Triggering Dopamine Development
tags:
  - TDD
excerpt: >-
  The other morning I attended an excellent talk by Andrew Richardson on the
  importance of having small goals. One of the points Andrew made was that
  completing a task, however small,  triggers a hit of Dopamine - the
  neurotransmitter associated with feelings of pleasure and satisfaction. The
  way Andrew described the feedback loop involved reminded me  of Test-Driven
  Development (TDD). This got me thinking - maybe this is why I like it so much?
date: 2023-05-17 12:44:16
---

The other morning I attended an excellent talk by [Andrew Richardson](https://www.linkedin.com/in/andrew-richardson-b3586819/) on the importance of having small goals. One of the points Andrew made was that completing a task, however small, triggers the release of Dopamine - the neurotransmitter associated with feelings of pleasure and satisfaction. This got me thinking about whether or not this could be a factor in Test-Driven Development (TDD).

One of the things I've always liked about TDD is the feeling of constantly making progress. The cycle of Red-Green-Refactor keeps me focused and every time a new test passes I feel like I’m getting somewhere. If one looks at this as a series of small tasks the Dopamine angle makes perfect sense. The process of creating a failing test, making it pass and finally refactoring is essentially setting and achieving small tasks. Dopamine FTW!

Making constant progress and feel-good chemicals aside there are some other benefits I've found with TDD:

**Confidence Building.** TDD allows me to try things and quickly rollback to a working state if they don’t work out as planned. If I see a potential refactor I can make it and get fast feedback about any side-effects. 
 
**Faster ‘Inner Loop’**.  TDD makes my Inner Loop faster because I spend negligible time in the debugger. Spinning up a test environment (taking minutes) to hit a break-point, look at some values, change a line of code and then do it again is painfully slow. With TDD I can generally avoid this.

> The ‘Inner Loop’ is "The iterative process of writing, building and debugging code that a single developer performs before sharing the code, either publicly or with their team.".
 
**Leaves a Rough Edge.** In pottery 'leaving a rough edge' means to leave a piece of obviously unfinished work so that you can get back to the flow of creating quickly. If I need to stop coding I will leave a failing test as a 'rough edge'. When I come back I start the failing test provides an obvious starting point. I also create stubs of failing tests as I go reminding me to cover edge cases. This means I have an executable task list. When all the ‘tasks’ are green then I am done. 
 
**Documentation.** I'm not a big fan of traditional documentation in the form of Confluence, Word Docs or Code Comments. As Ron Jeffries says "Code never lies, comments sometimes do". TDD creates executable documentation that explains the intended behavior of the code. This helps developers who come after (in many cases 'future me') to understand what I was trying to do. 

So if you're curious about TDD and/or want to get your daily Dopamine fix then why not give TDD a go? 