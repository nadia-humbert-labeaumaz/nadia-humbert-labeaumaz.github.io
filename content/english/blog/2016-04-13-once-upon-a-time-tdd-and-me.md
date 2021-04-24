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

Once upon a time, there was a young woman that had plenty of projects and passions and was a bit hyperactive. She doesn't really enjoy talking about her life and asks herself very seriously how is she going to write this post.
Among her early age dreams were learning many things about science, especially biosciences, and about software engineering. She started with biosciences (by the way, they are extremely fascinating, nothing is more complex and well crafted than the human body) and then she decided to continue with software engineering. However, the software engineering program that she completed only lasted one year (6 months of classes and 6 months of internship). Of course, this was only a door to access the world that she wanted to discover so much.

<!-- more -->

Not having much knowledge nor experience in IT, she was (and she still is) looking for methods, tools and good practices that would allow her to progress and produce clean code. Among these tools there is TDD.

TDD is a development tool that preconizes to write the tests before the production code. It was invented by Kent Beck. And you know what ? It is truly deeply deligthful ;-) (this sentence was a lot better in French, by the way). In this article, I am going to talk about my personal experience with TDD: the beginnings, the pros and cons and some ideas to set it up.

## The beginnings

At the beginning, I saw the theoretical interest of this tool and I was fascinated by the concept (I find it very clever). On the other hand, I thought that it made me work a bit slower and that it was a little complicated.
I tried it for the first time towards the end of my first year as a software developer and within a fairly complex project in terms of business, technologies and challenges. Needless to say that TDD was not the only thing that my small head had to process! As a consequence, I tried to use it when I was not overworked and as often as possible. Moreover, the large majority of developers of this big project did not use this tool. Thus, it was sometimes complicated to use TDD in this context but it was definitely out of the question to quit, I saw the potential of this tool and I knew that I had to persevere.

After this first year of experience in software development, during which I learned a lot, and a little experience in TDD, I decided to take time for some of my personal projects. And then, everything changed. Working on a project from scratch, with full latitude, I could use TDD more easily.

After a good night of sleep, I therefore started my own project with a test. It was so beautiful. At this very point, I understood a number of benefits that TDD can provide.

## Pros

### Express the business rather than doing an implementation

We focus on the business rather than on an implementation.

### Work unitarily and be more efficient

We only do one thing at a time and thus are more efficient. Contrary to what I thought at the beginning, I quickly realized that TDD allows to go faster. Indeed, we don't ask ourselves twenty questions at a time, even existential ones to create the best design. I am not saying that we shouldn't think anymore. By the way, I think we should and that it has to always be part of our job as software developers. On the other hand, TDD allows to ask ourselves even more relevant and targeted questions, a feature at a time.

### Have a test harness

By starting with tests, not only we are sure that we are going to have tests but also we know that they are going to be more relevant. Indeed, they cover real business needs.
This test harness permits to ensure that our code is protected and that, during refactoring or addition of new features, we are going to be able to detect regressions easily and quickly, unit testing being the least expensive in terms of implementation and allowing to have an extremely short feedback loop.

### Gain of confidence

By using TDD, I know I develop exactly what I need to, which is not negligible! This allowed me to gain confidence in my developments.

### Avoid bad design

TDD lets us know when our design is bad. Indeed, if we can't test our code base, it means that our design is no longer adapted or has never been. It is an alarm to encourage us to render it simpler and adapted to our needs of the current time. For example, if we have to mock and fix a behaviour to test another that depends on it but we can't, this probably means that our classes are strongly coupled. This can happen when we invoke a static method in a class, for instance. In this case, classes are strongly linked because we can't easily change the implementation of what is being used. However, classes should be strongly decoupled in order to make the system more maintainable (changeable and extensible by changing implementations easily).

Moreover, I could notice that the design that emerges by using TDD is rather simple. I think this is the result obtained by the overall benefits of TDD: focus on the business, perform one thing at a time and have more self-confidence result in the construction of a clean design.

## Cons

Currently, I don't see any and I have trouble trying to imagine some. By the way, I am very surprised to see that TDD is regularly misunderstood. It is often seen as a waste of time but I confirmed that this is only a false impression. There is obviously an adjustment period and it may vary depending on the developer. Nevertheless, I think that TDD is a good investment and deserves to be implemented. But how can we set it up ?

## Set up

It is never easy to try something new, but if we never challenged ourselves and if we never pushed our limits, we would miss many extraordinary things and we would get bored a lot too. I think that the way to set it up in a given project depends on its context, like for everything else in life, and we have to always adapt to it as good as possible. However, I will try to share some ideas on the subject here.

### Start with small goals

To be ambitious is great but it is hard to climb to the top of the mount Everest, the first time, without stopping. In the same way, during the set up of TDD our goals should be measured and well defined from the beginning. It could be as simple as encouraging all the team members to use TDD sometimes in the week. This way, they can progressively get familiar with the tool.

### Do not rush or be discouraged

From my personal experience, it is useless to want to assimilate the tool in two minutes and then judge whether it is suitable or not. Indeed, if everybody judged us in two minutes, they would have a perception of us not necessarily relevant nor complete. It is important to take the required time to know the tool, understand it and properly use it.

### Do not consider it as something outside developements

Like refactoring, TDD should, in my opinion, be a part of our developments. So it should not be something that we should ask the permission for but an integral part of our developments. As a consequence, we must take into account the adjustment period in our potential estimations, hence the importance of having small goals at first. After the adaptation period, it is quite possible that the estimations lower : I think TDD makes you go faster, for the reasons mentioned above, and we will no longer have to take into account the adaptation time.

### Make it a team effort

I do not think that designating some team members to test the tool is a good idea. The whole team should be equally involved in the process from the start. Otherwise, I think the chances to succeed diminish greatly. Indeed, if only a few members of the team are actually concerned by the tool, the others may lose interest on it. Furthermore, if the communication about the experience is not ideally done, the allocated time to it will be misused (the results may be null, some team members may be discouraged and those who had the "official permission" to test it may be disappointed). Finally, we risk to miss out on a good collaboration between coworkers and thus better results.

### Have good communication within the team

Don't hesitate to often talk about it. Different perspectives will enrich the experience. Nevertheless, do not forget my advice n°2 : do not be discouraged!

## Conclusion

TDD is a development tool that has a lot of advantages. It is not necessarily obvious to see them at first sight but you should not abandon the ship because the journey is really worth it! I hope you enjoyed this story and that it will be helpful to you. See you soon!
