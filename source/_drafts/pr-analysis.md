---
title: Optimizing Pull Request Effectiveness- A Metrics-Driven Approach
tags:
- metrics
- devops
category: Software Engineering
ogimage: pr3.png
date: 2022-05-01
excerpt:
    A discussion on how to evaluate the effectiveness of your Pull Request process using simple metrics.
---

> Disclaimer: In this post I will be talking about metrics related to the Pull Request process. These metrics cannot, and should never, be used to infer anything about the quality of the underlying code or the author. Furthermore teams should avoid setting targets against any of these metrics. <a href="https://en.wikipedia.org/wiki/Goodhart%27s_law">Doing so would distort any meaning or value that could be inferred from them</a>.

# Pull Requests
Most of our teams use Pull Requests (PR) to control the flow of code from one branch to another. Typically once someone has completed a piece of work the will create a Pull Request to merge this work into a branch which contains the shared work of the team.  At this point automated checks and tests can be run and the code is reviewed by one or more members of the team. Once everyone is satisfied the code is merged.

Pull Requests help to enforce the team's standards and identify issues within the code. Having multiple team members review the changes aids with collaboration and knowledge sharing. The main drawback though is that Pull Requests are a bottleneck. The author of the Pull Request needs to wait until one or more reviewers are available to have the code merged. This imposes a 'speed limit' on the team as a whole as changes cannot be released faster than the time it takes to complete the Pull Request process. If you combine this with poorly defined standards or an unclear review process the impact can be significant.

Given the potential impact it is crucial that teams employing Pull Requests are receiving the maximum value from the practice and minimise the associated context-switching and delays. This got me thinking about ways to quickly spot where a team may be struggling with their Pull Request process. Specifically I was wondering if we could use metrics to spot some common anti patterns.

# Pull Request Anti-Patterns

## Long Cycle Times
Pull Requests need to be dealt with quickly. When Pull Requests take a long time this delays feedback for the author and team as a whole resulting in context switching and rework.

## Rubber-stamping
Pull Requests are approved with no real consideration. PR's are simply a process to be followed. This creates the illusion of some form of Quality Process but none of the value.

## Super-sized PR's
Here PR's represent large pieces of work often containing multiple features or bug fixes. Consequently these behemoths take time to review or receive a less than thorough review.

## Unacknowledged Feedback
In this case reviewer comments do not result in any changes to the PR. This could be because the comments are not accepted and are being ignored. 

## Wheel-spinning
In some cases PR's require huge amounts of discussion and multiple rounds of amendments before they are accepted. In some cases this is legitimate but it can also point to a lack of clarity and concensus with regards to code standards and design.


# Evaluating Pull Request Effectiveness

Based on the anti-patterns about we can use 4 metrics to help us spot them.

1. Overall Duration:
Measure the time it takes for a PR to move from creation to merging. Short durations indicate swift code integration, while extended periods may highlight coordination challenges. Addressing these coordination issues can significantly enhance the team's efficiency. Also keep in mind that faster integration can also be a sign of rubber-stamping where the PR is accepted and merged without anyone really evaluating it.

2. Reviewers per PR:
Assess the number of reviewers assigned to each PR. A diverse set of reviewers contributes to improved code quality and knowledge sharing. Identify scenarios where having too few or too many reviewers could be indicative of potential issues. It's also important to consider who is performing the reviews. Is it always the same cohort? Are there people who never contribute to the reviews?

3. Commits per PR:
Analyze the number of commits in each PR and explore the correlation with the complexity of code changes. Smaller, more frequent commits often lead to more manageable PRs, facilitating a smoother review process. There is an point of diminishing returns with regards to code review effectiveness. A [2006 Study](https://static0.smartbear.co/support/media/resources/cc/book/code-review-cisco-case-study.pdf) showed that code review effectiveness dropped sharply after just 200 lines of code. Consequently, a Pull Request with a large number of commits covering hundreds of lines of code will take a long time to review thoroughly or, more likely, receive a less-than-thorough review.

4. Iterations per PR:
Examine the number of iterations a PR undergoes. Multiple iterations can indicate a healthy feedback loop, fostering continuous improvement. Excessive iterations, on the other hand, may suggest communication challenges or unclear coding standards.

It's clear from the above that you need to look at these 4 metrics holistically. Whilst we want the shortest duration possible we also want to have a balanced number and variety of reviewers. Furthermore we want PR's that are not too large - but also not so small to be trivial. Finally we would want to see some iterations - but not too many.

# Case Study
Let's look at a practical example.

<img src="pr1.png"/>

The first thing to notice here is the Duration - on average it's taking 33 Hours to close a Pull Request. This means that If I start work on a new feature on Monday morning and submit the Pull Request later that day I would only get feedback at best Friday or Monday! If I continue with new features in the meantime I would be heading for significant context switching the following week as all my PR's reviews start to come back.

The next thing to check would be the number of Iterations. The figure of 1 indicates that on-average Pull Requests are accepted with no revisions. This could indicate a mature and stable team but could also indicate a lack of rigor in the review process. If you combine this with the 33 Hour wait time what we are seeing is an enforced delay with very little chance of valuable feedback.

<img src="pr2.png"/>

In terms of the ratio of Author to Reviewer the entire team participates in the review process. There are some people though who spend more time reviewing than authoring. In some cases people are exclusively Reviewers. As to what this indicates would be dependent on your team. You have to ask yourself, "Does it make sense that this person is not creating any Pull Requests?". Practices such as Pairing or reviews by Architects or Engineering Leads may distort the picture so you must look at this holistically. 

<img src="pr3.png" width="500px"/>

In this view you can see the impact of the long review process. Note how the majority of the PR's are closed on Monday or Friday. In fact what could be happening here is that the team are optimising to reduce context-switching by saving up reviews for these days.

# Tooling
Collecting these stats is not trivial - especially if you work with multiple teams and repos. I've created an extension for Azure DevOps to automate the process. You can get the extension [here](https://marketplace.visualstudio.com/items?itemName=CodeMk.codemk-pullrequest).

I'll also be releasing a stand-alone version for GitHub in there's enough interest.

<img src="pr4.png"/>

---
# Pull Requests
Pull Requests are used in some teams to control the flow of code from one branch to another. Typically a developer will complete a piece of work on one branch and then create a Pull Request to merge this work into a branch which contains the shared work of the team. At this point automated checks and tests can be run and the code is reviewed by one or more members of the team. Once everyone is satisfied the code is merged.

Pull Requests help to enforce the team's standards and identify issues within the code. Having multiple team members review the changes aids with collaboration and knowledge sharing. The main drawback though is that Pull Requests are a bottleneck. The author of the Pull Request needs to wait until one or more reviewers are available to have the code merged. This imposes a 'speed limit' on the team as a whole as changes cannot be released faster than the time it takes to complete the Pull Request process. If you combine this with poorly defined standards or an unclear review process the impact can be significant.

Given the potential impact it is crucial that teams employing Pull Requests are receiving the maximum value from the practice and minimise the associated context-switching and delays. In the following sections, we'll explore key metrics that can provide teams with actionable insights to enhance their Pull Request process, ensuring efficient collaboration and timely code integration.

# Evaluating Pull Request Effectiveness

Let's propose 4 simple metrics as a starting point for evaluating Pull Request Effectiveness:

1. Overall Duration:
Measure the time it takes for a PR to move from creation to merging. Short durations indicate swift code integration, while extended periods may highlight coordination challenges. Addressing these coordination issues can significantly enhance the team's efficiency. Also keep in mind that faster integration can also be a sign of rubber-stamping where the PR is accepted and merged without anyone really evaluating it.

2. Reviewers per PR:
Assess the number of reviewers assigned to each PR. A diverse set of reviewers contributes to improved code quality and knowledge sharing. Identify scenarios where having too few or too many reviewers could be indicative of potential issues. It's also important to consider who is performing the reviews. Is it always the same cohort? Are there people who never contribute to the reviews?

3. Commits per PR:
Analyze the number of commits in each PR and explore the correlation with the complexity of code changes. Smaller, more frequent commits often lead to more manageable PRs, facilitating a smoother review process. There is an point of diminishing returns with regards to code review effectiveness. A [2006 Study](https://static0.smartbear.co/support/media/resources/cc/book/code-review-cisco-case-study.pdf) showed that code review effectiveness dropped sharply after just 200 lines of code. Consequently, a Pull Request with a large number of commits covering hundreds of lines of code will take a long time to review thoroughly or, more likely, receive a less-than-thorough review.

4. Iterations per PR:
Examine the number of iterations a PR undergoes. Multiple iterations can indicate a healthy feedback loop, fostering continuous improvement. Excessive iterations, on the other hand, may suggest communication challenges or unclear coding standards.

It's clear from the above that you need to look at these 4 metrics holistically. Whilst we want the shortest duration possible we also want to have a balanced number and variety of reviewers. Furthermore we want PR's that are not too large - but also not so small to be trivial. Finally we would want to see some iterations - but not too many.

# Applying the metrics

There are a number of common anti-patterns associated with Pull Requests:
 - Rubber-Stamping: PRs routinely accepted without proper consideration     
 - Lone Ranger Syndrome: Single developer submits PRs without seeking collaboration   
 - Long Review Queues:   PRs spend extended time in the review queue  
 - Excessive Changes:    PRs have a large number of changes, making reviews complex
 - Inconsistent Coding Standards: Frequent violations of coding standards within PRs   
 - Unacknowledged Feedback: Developers submit PRs without addressing reviewer feedback

 In the absence of metrics these can be tricky to spot unless you are actively monitoring each Pull Request. Teams often drift into these patterns over time so they may not even be aware that they are happening. Using the metrics though can quickly highlight potential issues. Each anti-pattern will show up in the metrics as follows:

| Anti-Pattern               | Duration           | Commit Count       | Iteration Count    | Reviewer Count     |
|----------------------------|--------------------|--------------------|--------------------|---------------------|
| Rubber-Stamping            | Low                | Varies                | Low                | Low                |
| Lone Ranger Syndrome       | Short to Moderate   | Low                | Low                | Low                |
| Long Review Queues         | Prolonged          | Varies             | Varies             | Low                |
| Excessive Changes          | Varies             | High               | Varies             | Low                |
| Inconsistent Coding Standards | Varies             | Varies             | High               | Varies             |
| Unacknowledged Feedback     | Varies             | Low                | Low                | Varies             |

Once you have identified something that you would want to focus on you can use the metrics to evaluate the impact of any changes you make to your process.

# Avoiding the Pitfalls
These metrics have been useful in identifying process bottlenecks in teams but they can cause real damage is misused. Here are some tips to get the most value of of them.

1. Quantification, Not Targeting:
These metrics should not be treated as rigid targets. Instead, they serve as quantifiable indicators to identify areas for improvement. Teams should focus on continuous improvement rather than striving for specific numerical goals.

2. Context Matters:
Acknowledge that the effectiveness of PRs varies based on project size, team dynamics, and the nature of the codebase. Encourage teams to consider these contextual factors when interpreting and acting upon the metrics.

3. Strategies for Improvement:
Provide practical examples of how teams have successfully improved their PR process based on these metrics. Highlight strategies such as streamlined coordination, diversified reviewer assignments, and effective feedback mechanisms.

# Case Study
Let's look at a practical example.

<img src="pr1.png"/>

The first thing to notice here is the Duration - on average it's taking 33 Hours to close a Pull Request. This means that If I start work on a new feature on Monday morning and submit the Pull Request later that day I would only get feedback at best Friday or Monday! If I continue with new features in the meantime I would be heading for significant context switching the following week as all my PR's reviews start to come back.

The next thing to check would be the number of Iterations. The figure of 1 indicates that on-average Pull Requests are accepted with no revisions. This could indicate a mature and stable team but could also indicate a lack of rigor in the review process. If you combine this with the 33 Hour wait time what we are seeing is an enforced delay with very little chance of valuable feedback.

<img src="pr2.png"/>

In terms of the ratio of Author to Reviewer the entire team participates in the review process. There are some people though who spend more time reviewing than authoring. In some cases people are exclusively Reviewers. As to what this indicates would be dependent on your team. You have to ask yourself, "Does it make sense that this person is not creating any Pull Requests?". Practices such as Pairing or reviews by Architects or Engineering Leads may distort the picture so you must look at this holistically. 

<img src="pr3.png" width="500px"/>

In this view you can see the impact of the long review process. Note how the majority of the PR's are closed on Monday or Friday. In fact what could be happening here is that the team are optimising to reduce context-switching by saving up reviews for these days.

# Tooling
Collecting these stats is not trivial - especially if you work with multiple teams and repos. I've created an extension for Azure DevOps to automate the process. You can get the extension [here](https://marketplace.visualstudio.com/items?itemName=CodeMk.codemk-pullrequest).

I'll also be releasing a stand-alone version for GitHub in there's enough interest.

<img src="pr4.png"/>

