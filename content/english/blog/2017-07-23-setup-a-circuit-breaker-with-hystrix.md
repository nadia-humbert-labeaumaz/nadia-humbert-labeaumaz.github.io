---
type: "post"
title: "Setup a Circuit Breaker with Hystrix, Feign Client and Spring Boot"
slug: "setup-a-circuit-breaker-with-hystrix"
date: 2017-07-23 13:33:38 +0200
description: ""
author: Nadia Humbert-Labeaumaz
image : "images/blog/circuit-breaker/header.jpg"
bg_image: "images/blog/circuit-breaker/header.jpg"
categories: ["Software"]
tags: ["Java", "Software Design"]
draft: false
comments: true
---

In a microservices architecture, several things could go wrong. For example, a middleware, the network or the service you want to contact could be down. In this world of uncertainty, you have to anticipate problems to avoid breaking the entire chain and throwing an error to the end-user when you could offer a partially degraded service instead.

This article aims to show how to implement the circuit breaker pattern using Hystrix, Feign Client and Spring Boot.

<!--more-->

## Feign Client Crash Course

[Feign](https://github.com/OpenFeign/feign) is an HTTP client created by Netflix to make HTTP communications easier. It is integrated into Spring Boot with the `spring-cloud-starter-feign` starter.

To create a client to consume an HTTP service, creating an interface annotated with `@FeignClient` is necessary. Endpoints can be declared in this interface using an API close to the Spring MVC API. It is also required to add The `@EnableFeignClients` annotation to a Spring Configuration class.

```java
@Configuration
@EnableFeignClients
public class FeignConfiguration {

}
```

```java
@FeignClient(name = "videos", url = "http://localhost:9090/videos")
public interface VideoClient {

    @PostMapping(value = "/api/videos/suggest")
    List<Suggestion> suggest(@RequestBody ViewingHistory history);

}
```

An instance of `VideoClient` is automagically injected into the Spring application context and can be autowired and used throughout the application. Moreover, if the `videos` microservice is registered to the same discovery service as the current microservice, there is no need for an URL as it will be retrieved for you based on the `name`.

If the `videos` service, a middleware or the network happens to be down or overloaded, the `suggest` method will throw a `FeignException` that will be propagated throughout the stack if not caught.

## Create a Fallback Implementation

Fortunately, Spring Cloud comes with a solution to this problem: a circuit breaker. In this article, we will use [Hystrix](https://github.com/Netflix/Hystrix). It was also created by Netflix and also integrated into Spring Boot using the `spring-cloud-starter-hystrix` starter.

The idea is to create an implementation of the `VideoClient` and mark it as the default behaviour if `videos` is unreachable or overloaded. Like a lot of other Spring features, it is enabled using an annotation: `@EnableCircuitBreaker`.

```java
@Configuration
@EnableFeignClients
@EnableCircuitBreaker
public class FeignConfiguration {

}
```

```java
@FeignClient(name = "videos", url = "http://localhost:9090/videos", fallback = VideoClientFallback.class)
public interface VideoClient {

    @PostMapping(value = "/api/videos/suggest")
    List<Suggestion> suggest(@RequestBody ViewingHistory history);

}
```

```java
@Component
public class VideoClientFallback implements VideoClient {

    @Override
    public List<Suggestion> suggest(ViewingHistory history) {
      // Degraded service: no suggestion to offer
      return new ArrayList<>();
    }

}
```

A configuration property has to be added to the `application.yml` file of the Spring Boot application to tell Feign to enable Hystrix.

```yml
feign:
    hystrix:
        enabled: true
```

Voilà! Whenever the remote service is unavailable, the `suggest` method of the `VideoClientFallback` will be called, and the end-user will not get an error violently thrown at her.

## Keep Track of the Source Error

With this setup, the fallback will be called regardless of the initial error. If you want to retrieve this error and do something with it, you can use a `FallbackFactory`.

```java
@FeignClient(name = "videos", url = "http://localhost:9090/videos", fallbackFactory = VideoClientFallbackFactory.class)
public interface VideoClient {

  @PostMapping(value = "/api/videos/suggest")
  List<Suggestion> suggest(@RequestBody ViewingHistory history);

}
```

```java
@Component
public class VideoClientFallbackFactory implements FallbackFactory<VideoClient> {

    @Override
    public VideoClient create(Throwable throwable) {
        return new VideoClientFallback(throwable);
    }

}
```

```java
public class VideoClientFallback implements VideoClient {

    private final Throwable cause;

    public VideoClientFallback(Throwable cause) {
      this.cause = cause;
    }

    @Override
    public List<Suggestion> suggest(ViewingHistory history) {
        if (cause instanceof FeignException && ((FeignException) cause).status() == 404) {
            // Treat the HTTP 404 status
        }

        return new ArrayList<>();
    }

}
```

## Conclusion

Microservices foster low coupling between components and resiliency. Hence, throwing an error every time a middleware or service is down would be unfortunate. The circuit breaker pattern explained in this article allows you to ensure the continuity of service. As often, Spring Boot is a great help to set up this mechanism readily.
