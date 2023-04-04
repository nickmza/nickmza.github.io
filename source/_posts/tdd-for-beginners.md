---
title: 'TDD - Advice for Beginners'
date: 2016-03-21 13:57:01
tags: 
- TDD
---

The other day I read an article by Ian Sommerville describing his experiences with Test Driven Development (TDD) and his ultimate conclusion that TDD is fundamentally flawed. Robert Martin quickly posted an excellent response pointing out that what Sommerville was experiencing was typical of people new to Test Driven Development. Whilst Martin's response pulls no punches in pointing out the source of Sommerville's issues it does not really leave any practical advice for someone struggling with learning TDD. 

Sommerville's article struck a chord with me because what he was describing was so similar to my own experiences whilst learning TDD. Brittle tests and the feeling that somehow I was perverting the code in order to pass a test haunted me to the point that I nearly came to same conclusion as Sommerville. Ultimately though these issues were, as Martin points out, symptoms of being an amateur. I kept going and many, many tests later have got to the point where I am comfortable with TDD.

My advice for someone struggling with TDD would be as follows:

## Take Smaller Steps

Sommerville describes TDD as follows: "Test-first or test-driven driven development (TDD) is an approach to software development where you write the tests before you write the program". This is not an accurate description of TDD. Many developers reading this will conclude that you need to write the whole test first, before you write any production code. This simple misunderstanding of the process of TDD is at the core of many of the issues people have with TDD. 

When you write the whole test first you are robbing yourself of the ability to learn from the process of TDD. The key phrase here is Test DRIVEN. The Tests DRIVE the development. If you write the whole test first you are making a number of untested assumptions about how the code will play out and then forcing the production code to fit in with those assumptions. This leads to many of the issues that Sommerville describes.

## You need to take smaller steps. 

Try this: Write enough test code to fail the test (and a compilation error is a failing test). Write enough production code to make the test pass or clear the compilation error. Refactor the code you have just written and repeat the process until the unit test passes completely. For a short description read The Three Rules of TDD. For a detailed one read Kent Beck's TDD By Example.

Many developers balk at this process. It feels unnatural. Which, if you have been happily coding for years, it would be. You would not always apply TDD like this but for learning TDD it really helps ingrain the cadence and cognitive processes involved. What I find is that the more uncertain I am of the design the smaller the increment I work in - almost down to a 1:1 ratio of test to production code. As the design emerges and I become more confident I move in larger steps. But never straying from the basic pattern. All production code is written in the service of passing a test. This ensures that I only write code I need. 

When I find people struggling with tests it is often because they have broken out of the pattern and are implementing reams of production code. The impact of this on the TDD process is that by the time you return to the test the production code may be completed. You then need to retrofit the test to match your production code. At this point the Test is not Driving the development it is a Passenger. Tests written in this fashion are typically brittle and the number of tests written for a piece of functionality is generally less. 

## Get a Coach

Becoming proficient at TDD is not easy. It is a skill and, as with all skills, it takes time and effort to master. This is where I think many developers, especially experienced ones, go wrong. They expect to pick up TDD after reading a couple of articles and looking at some example tests. Their initial enthusiasm is soon extinguished by the frustrations of learning TDD and ultimately they give up all together. 

It makes a huge difference to have someone who has done this before helping you. To point out alternatives or show you how to make the design more testable. It also helps to keep the enthusiasm up. If everyone in the team is new to TDD it is easy for any frustrations to boil over into defeatism. 

## Practice, Practice, Practice

Trying to apply TDD for the first time to your current or new project at work is difficult. There are all sorts of other factors that complicate the process - time, budget and legacy code all conspire against the application of TDD. I often here this: "We started with TDD but it was taking so long our project manager said that we don't have time for it.". 

My suggestion is to become proficient at the basic process of TDD before trying to apply it to a real-world scenario. There are hundreds of practice examples, also called Katas, for TDD. If you can't do a basic kata such as String Calculator then you are not ready to start applying this at work. 

Another great way to learn is at code camps and hackathons. Here you will be exposed to different views and approaches to TDD in a fun and open environment. It's a great way to confirm and extend your own views.

## Conclusion

The response to novice frustrations sometimes tends to 'If it is not working then you are not doing it right'. This is not constructive and, in the absence of any constructive advice, can be construed as zealotry. I've outlined here some of the things that helped me with TDD and hope it will help you as well.