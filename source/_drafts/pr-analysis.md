---
title: Optimizing Pull Request Effectiveness- A Metrics-Driven Approach
tags:
---

> Disclaimer: In this post I will be talking about metrics related to the Pull Request process. These metrics cannot, and should never be used to, infer anything about the quality of the underlying code or the author. Furthermore teams should avoid setting targets against any of these metrics. <a href="https://en.wikipedia.org/wiki/Goodhart%27s_law">Doing so would distort any meaning or value that could be inferred from them</a>.

# Pull Requests
Pull Requests are used in some teams to control the flow of code from one branch to another. Typically a developer will complete a piece of work on one branch and then create a Pull Request to merge this work into a branch which contains the shared work of the team. At this point automated checks and tests can be run and the code is reviewed by one or more members of the team. Once everyone is satisfied the code is merged.

Pull Requests help to enforce the team's standards and identify issues withing the code. Having multiple team members review the changes aids with collaboration and knowledge sharing. There are some short comings however. Pull Requests are a bottleneck. The author of the Pull Request needs to wait until one or more reviewers are available to have to code merged. This imposes a 'speed limit' on the team as a whole as changes cannot be released faster than the time it takes to complete the Pull Request process.

Given these issues it is crucial that teams employing Pull Requests are receiving the maximum value from the practice and minimise the associated context-switching and delays. In the following sections, we'll explore key metrics that can provide teams with actionable insights to enhance their Pull Request process, ensuring efficient collaboration and timely code integration.

# Evaluating Pull Request Effectiveness

We propose 4 simple metrics as a starting point for evaluating Pull Request Effectiveness:

1. Overall Duration:
Measure the time it takes for a PR to move from creation to merging. Short durations indicate swift code integration, while extended periods may highlight coordination challenges. Addressing these coordination issues can significantly enhance the team's efficiency. Also keep in mind that faster integration can also be a sign of rubber-stamping where the PR is accepted and merged without anyone really evaluating it.

2. Reviewers per PR:
Assess the number of reviewers assigned to each PR. A diverse set of reviewers contributes to improved code quality and knowledge sharing. Identify scenarios where having too few or too many reviewers could be indicative of potential issues. It's also important to consider who is performing the reviews. Is it always the same cohort? Are there people who never contribute to the reviews?

3. Commits per PR:
Analyze the number of commits in each PR and explore the correlation with the complexity of code changes. Smaller, more frequent commits often lead to more manageable PRs, facilitating a smoother review process. There is an point of diminishing returns with regards to code review effectiveness. A [2006 Study](https://static0.smartbear.co/support/media/resources/cc/book/code-review-cisco-case-study.pdf) showed that code review effectiveness dropped sharply after 200 lines of code. Consequently a Pull Request with a large number of commits covering hundreds of lines of code will take a long time to review thoroughly or, more likely, receive a less than thorough review.

4. Iterations per PR:
Examine the number of iterations a PR undergoes. Multiple iterations can indicate a healthy feedback loop, fostering continuous improvement. Excessive iterations, on the other hand, may suggest communication challenges or unclear coding standards.

It's clear from the above that you need to look at these 4 metrics holistically. Whilst we want the shortest duration possible we also want to have a balanced number and variety of reviewers. Furthermore we want PR's that are not too large - but also not so small to be trivial. Finally we would want to see some iterations - but not too many.

# Applying the metrics

Let's take a look at how these metrics could play out in reality by showing how they could highlight common Pull Request anti-patterns.
 - Rubber-Stamping: PRs routinely accepted without proper consideration     
 - Lone Ranger Syndrome: Single developer submits PRs without seeking collaboration   
 - Long Review Queues:   PRs spend extended time in the review queue  
 - Excessive Changes:    PRs have a large number of changes, making reviews complex
 - Inconsistent Coding Standards: Frequent violations of coding standards within PRs   
 - Unacknowledged Feedback: Developers submit PRs without addressing reviewer feedback

| Anti-Pattern               | Duration           | Commit Count       | Iteration Count    | Reviewer Count     |
|----------------------------|--------------------|--------------------|--------------------|---------------------|
| Rubber-Stamping            | Low                | Low                | Low                | Low                |
| Lone Ranger Syndrome       | Short to Moderate   | Low                | Low                | Low                |
| Long Review Queues         | Prolonged          | Varies             | Varies             | Low                |
| Excessive Changes          | Varies             | High               | Varies             | Low                |
| Inconsistent Coding Standards | Varies             | Varies             | High               | Varies             |
| Unacknowledged Feedback     | Varies             | Low                | Low                | Varies             |


# Avoiding the Pitfalls
1. Quantification, Not Targeting:
These metrics should not be treated as rigid targets. Instead, they serve as quantifiable indicators to identify areas for improvement. Teams should focus on continuous enhancement rather than striving for specific numerical goals.

2. Context Matters:
Acknowledge that the effectiveness of PRs varies based on project size, team dynamics, and the nature of the codebase. Encourage teams to consider these contextual factors when interpreting and acting upon the metrics.

3. Strategies for Improvement:
Provide practical examples of how teams have successfully improved their PR process based on these metrics. Highlight strategies such as streamlined coordination, diversified reviewer assignments, and effective feedback mechanisms.

# Case Study
Here's an example from a fictional team.


<img src="pr1.png"/>

<img src="pr2.png"/>

<img src="pr3.png"/>


# Tooling


