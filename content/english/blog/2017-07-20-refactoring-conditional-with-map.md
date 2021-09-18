---
type: "post"
title: "Refactoring Conditional Structures with Map"
slug: "refactoring-conditional-with-map"
date: 2017-07-20 15:32:15 +0100
description: ""
author: Nadia Humbert-Labeaumaz
image : "images/blog/refactoring-conditional/header.jpg"
bg_image: "images/blog/refactoring-conditional/header.jpg"
categories: ["Software"]
tags: ["Software Design", "Refactoring", "Java"]
draft: false
comments: true
---

When working on already existing codebases, I often encounter pieces of code that look like this:

```java
public class Day {
  public void start(Weather weather) {
    switch(weather) {
      case RAINY:
          takeAnUmbrella();
          break;
      case SUNNY:
          takeAHat();
          break;
      case STORMY:
          stayHome();
          break;
      default:
          doNothing();
          break;
    }
  }
}
```

Basically, depending on the weather, a specific action has to be taken. This kind of code is pretty hard to test and maintain. The goal of this article is to refactor it using a `Map`.

<!--more-->

## What is the Problem?

Using conditional structures like this might be a sign of bad design. Indeed, this code tends to grow indefinitely as new cases have to be handled, and the same code has to be modified repeatedly. So, inevitably, a time will come when the code will be so bloated that it will be tough to add new behaviour. This is a violation of the _Open Closed Principle_ which stipulates that the code should be open for extension but closed for modification. In other words, you should be able to add new behaviour to your code without modifying it.

## Transform the Imperative Algorithm into Data

By analyzing this code, it becomes clear that this algorithm is no more than a `Map`: for each weather (the key), a piece of code has to be executed (the value). Therefore, it is possible to perform a first refactoring iteration to make this conceptual `Map` concrete:

```java
public class Day {

  private final Map<Weather, Runnable> startOfTheDayActions = new HashMap<>();

  public Day() {
    startOfTheDayActions.put(Weather.RAINY, this::takeAnUmbrella);
    startOfTheDayActions.put(Weather.SUNNY, this::takeAHat);
    startOfTheDayActions.put(Weather.STORMY, this::stayHome);
  }

  public void start(Weather weather) {
    startOfTheDayActions.getOrDefault(weather, this::doNothing).run();
  }
}
```

The code is now already much clearer. Indeed, the mapping between the weather and the action to perform is explicit, and it is not necessary to modify the `start` method very often: each new case only requires a new entry in the `Map`.

Nonetheless, this do not solve all issues. In fact, the class still has to be modified to add a new entry. So, to go further, the `Map` can be passed as a parameter of the constructor.

```java
public class Day {

  private final Map<Weather, Runnable> startOfTheDayActions = new HashMap<>();

  public Day(Map<Weather, Runnable> startOfTheDayActions) {
    this.startOfTheDayActions = startOfTheDayActions;
  }

  public void start(Weather weather) {
    startOfTheDayActions.getOrDefault(weather, this::doNothing).run();
  }
}
```

Now, the only responsibility of the class is to use the mapping to perform the correct action. This mapping is now the responsibility of another class.

## Note about Spring Framework

If you are using Spring Framework and the `Day` class is a `@Component`, you can simply inject the `Map` as any other dependency.

```java
@Component
public class Day {

  private final Map<Weather, Runnable> startOfTheDayActions = new HashMap<>();

  public Day(@Qualifier("startOfTheDayActions") Map<Weather, Runnable> startOfTheDayActions) {
    this.startOfTheDayActions = startOfTheDayActions;
  }

  public void start(Weather weather) {
    startOfTheDayActions.getOrDefault(weather, this::doNothing).run();
  }
}
```

```java
@Configuration
public class ActionConfig {

  @Bean("startOfTheDayActions")
  public Map<Weather, Runnable> startOfTheDayActions() {
    Map<Weather, Runnable> actions = new HashMap<>();
    // Create mapping
    return actions;
  }
}
```

## Conclusion

This refactoring is easy to perform, but it can reduce the complexity of a method in a very efficient way. Indeed, I think the code should reveal intention and not be bloated with conditional structures when unnecessary.
