---
type: "post"
title: "Once upon a time TDD... and me"
slug: "once-upon-a-time-tdd-and-me"
date: 2016-04-13 23:56:11 +0200
description: ""
author: Nadia Humbert-Labeaumaz
image : "images/blog/tdd-story/header.jpg"
bg_image: "images/blog/tdd-story/header.jpg"
categories: ["Software"]
tags: ["Software Testing"]
draft: false
comments: true
---

Once upon a time, a young woman had plenty of projects and passions and was a bit hyperactive. She doesn't really enjoy talking about her life and asks herself, very seriously, how she will write this post.

Among her early age dreams were learning many things about science and software engineering. She started with biosciences (by the way, they are fascinating, nothing is more complex and well crafted than the human body) and then she decided to continue with software engineering. However, the software engineering program she completed only lasted one year (6 months of classes and 6 months of internship). Of course, this was only a door to access the world that she wanted to discover so much.

<!--more-->

Not having much knowledge nor experience in IT, she was (and still is) looking for principles, good practices, methods and tools that would allow her to learn and produce clean code. Among these tools, there is TDD.

TDD is a development tool that preconizes writing the tests before the production code. Kent Beck invented it. And you know what? It is truly deeply delightful ;-) (this sentence was a lot better in my head in French, by the way). In this article, I will talk about my personal experience with TDD: the beginnings, the pros and cons and some ideas to set it up.

## The Beginnings

In the beginning, I saw the theoretical interest of this tool, and I was fascinated by the concept (I find it very clever). But, on the other hand, I thought it made me work a bit slower, and it was a little complicated.
I tried it for the first time towards the end of my first year as a software developer and within a fairly complex project in terms of business, technologies and challenges. Needless to say that TDD was not the only thing that my head had to process! Consequently, I tried to use it when I was not overworked and as often as possible. Moreover, the large majority of developers of this big project did not use this tool. Thus, it was sometimes difficult to use TDD in this context, but it was definitely out of the question to quit, I saw its potential, and I knew that I had to persevere.

After this first year of experience in software development – during which I learned a lot – and a little experience in TDD, I decided to take time for my personal projects. And then, everything changed. By working on a project from scratch with full latitude, I could use TDD more readily.

Therefore, after a good night of sleep, I decided to start my own project with a test. It was so beautiful. At this very point, I understood several benefits that TDD could provide.

## Pros

### Expressing the Business Rather Than Doing an Implementation

We focus on the business rather than on implementation.

### Working by Units and Being More Efficient

We only do one thing at a time and thus are more efficient. Contrary to what I thought at the beginning, I quickly realized that TDD allows going faster. Indeed, we don't ask ourselves twenty questions at a time to create the best design. Of course, I am not saying that we shouldn't think anymore. By the way, I think it is an essential part of our job as software engineers. However, TDD allows us to ask ourselves even more relevant and targeted questions, a feature at a time.

### Building a Relevant Test Harness

By starting with tests, we build a relevant test harness. Indeed, the tests we write cover real business needs.
As a result, this harness ensures easy and quick detection of regressions during the addition of new features or refactoring – unit testing being the least expensive in terms of implementation and enabling an extremely short feedback loop.

### Gaining Confidence

Using TDD, I know I develop exactly what I need to – which is not negligible – letting me gain confidence in my code.

### Avoiding Bad Design

TDD lets us know when our design is bad. Indeed, if we can't test our codebase, it means that our design is no longer adapted or has never been. Therefore, it is an alarm to encourage us to render it simpler and tailor it to our needs. For example, if we have to mock a behaviour to test another that depends on it, it probably means that our classes are strongly coupled. Furthermore, decoupling your classes make your system more maintainable (i.e. changeable and extensible).

I also noticed the design emerging from TDD was clean and elegant as a result of the tool's overall benefits mentioned above: focusing on the business, performing one thing at a time and having increased self-confidence.

## Cons

Currently, I don't see any. Unfortunately, TDD is regularly misunderstood and perceived as a waste of time. This statement is a false impression that probably stems from the required adjustment period, which may vary for each developer. Nevertheless, TDD proves to be a good investment that deserves to be considered. But how can we set it up?

## Setting It Up

The way to set it up in a given project depends on its context. However, I will try to share some ideas on the subject here.

### Start With Small Goals

To be ambitious is great, but it is hard to climb to the top of Mount Everest for the first time without stopping. In the same way, during the setup of TDD, we should properly define and measure our goals from the beginning. For example, it could be as simple as encouraging all the team members to use TDD a few times in the week to become progressively familiar with the tool.

### Do Not Rush or Get Discouraged

From my personal experience, it is counterproductive to try to assimilate the tool and judge whether it is suitable or not in a few minutes. Indeed, if someone judged us in two minutes, the way they would perceive us would not necessarily be relevant nor complete. It is important to take the required time to know, understand and properly use the tool.

### Do Not Consider It as Something Outside Developments

Like refactoring, TDD should, in my opinion, be a part of our developments. So, it should not be something we should ask permission for but an integral part of our job. Consequently, we must take into account the adjustment period in our potential estimations, hence the importance of having small goals at first. After the adaptation period, the estimations likely become lower. Indeed, TDD makes us ultimately go faster for the reasons mentioned above, and we no longer have to factor in the extra time.

### Have Good Communication Within the Team

Don't hesitate to often talk about it. Different perspectives will enrich the experience.

Nevertheless, remember my advice n°2: don't get discouraged in the process.

## Conclusion

TDD is a development tool with many advantages, including clean design and fewer bugs and regressions. It is not necessarily easy to identify these benefits at first sight, but you should not abandon the ship because the journey is really worth it.

Thanks so much for reading; I hope you enjoyed it :-)
