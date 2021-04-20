---
type: "post"
title: "Refactoring Conditional Structures with Map"
date: 2017-07-20 15:32:15 +0100
description: ""
author: Nadia Humbert-Labeaumaz
image : "images/blog/refactoring-conditional/header.jpg"
bg_image: "images/blog/refactoring-conditional/header.jpg"
categories: ["Software"]
tags: ["Design", "Refactoring", "Java"]
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

Basically, depending on the weather, an action has to be taken. This kind of code is pretty hard to test and to maintain. The goal of this article is to refactor it using a `Map`.

<!-- more -->

## What is the Problem?

Using conditional structures like this might be a sign of bad design. Indeed, this code tends to grow indefinitely as new cases have to be handled and the same code has to be modified over and over. A time will come when the code will be so bloated that it will be very hard to add new behaviour. This is a violation of the _Open Closed Principle_ which stipulates that the code should be open for extension but closed for modification: you should be able to add new behaviour to your code without modifying it.

## Transform the Imperative Algorithm into Data

By analyzing this code, it becomes clear that this algorithm is no more than a `Map`: for a certain weather (the key), a piece of code has to be executed (the value). A first refactoring can be done to make this conceptual `Map` concrete:

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

This is a first step and the code is already much clearer. Now, the mapping between the weather and the action to perform is explicit and the `start` method will not have to be modified very often. When a new case must be handled, it's just a new entry in the `Map`.

Nonetheless, this do not solve all problems. The class still has to be modified to add a new entry. To go further, the `Map` can be passed as a parameter of the constructor.

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

Now the only responsibility of the class is to use the mapping to perform the correct action. This mapping is now the responsibility of another class.

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

This refactoring is very easy to do but it can reduce the complexity of a method in a very efficient way. I think that the code should reveal intention and should not be bloated with conditional structures when it is not necessary.
