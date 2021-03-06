---
type: "post"
title: "Testing Spring MVC Controllers"
slug: "testing-spring-mvc-controllers"
description: ""
date: 2015-10-31 12:54:08 +0100
author: Nadia Humbert-Labeaumaz
image : "images/blog/testing-spring-controllers/header.jpg"
bg_image: "images/blog/testing-spring-controllers/header.jpg"
categories: ["Software"]
tags: ["Java", "Software Testing"]
draft: false
comments: true
---

Since Spring 3.2 (January 2013), it has been possible to test Spring MVC controllers without an external framework.
This article aims to show how to test Spring MVC controllers using only Spring testing capabilities.

<!--more-->

To do so, a simple Spring Boot project will be used as support. You can find it on [GitHub](https://github.com/nphumbert/demo-test-spring-mvc). First, we will present the controller to test. Then, we will provide explanations on how to test it.

## Controller

The controller to test is shown below. It enables to perform a search in a fruit list.

```java
@Controller
public class ApplicationController {

    @RequestMapping(value = "/fruits", method = RequestMethod.GET)
    public String getFruits(@RequestParam(value = "search", required = false) String search, final Model model) {
        model.addAttribute("fruitBowl", search(search));
        return "fruits";
    }

    private List<String> search(String search) {
        if (StringUtils.isEmpty(search)) {
            return fruitBowl();
        }

        return fruitBowl().stream()
                .filter(fruit -> fruit.startsWith(search))
                .collect(toList());
    }

    private List<String> fruitBowl() {
        return asList("banana", "orange");
    }
}
```

## Test

It is necessary to annotate the test class as follows:

- `@RunWith(SpringJUnit4ClassRunner.class)` to benefit from Spring features in JUnit tests.
- `@SpringApplicationConfiguration(classes = DemoTestSpringMvcApplication.class)` to specify the configuration class that will be used during the test.
- `@WebAppConfiguration` to indicate that the Spring application context to load is a `WebApplicationContext`.

The entry point used to perform the tests is the class `MockMvc`. Thereafter, we will explain the usage of this class.

### Set up

It is possible to set up the `MockMvc` class as follows:

```java
@Before
public void setUp() {
    ApplicationController controller = new ApplicationController(); // 1
    this.mockMvc = MockMvcBuilders.standaloneSetup(controller).build(); // 2
}
```

First, the controller must be instantiated (`1`). Then, the mock is initialized using the static method `MockMvcBuilders.standaloneSetup()` (`2`). At this point, the mock is ready to test the controller.  

### Perform

The method `MockMvc.perform()` allows simulating HTTP requests to the controller. For instance, a GET request is made as follows:

```java
mockMvc.perform(
    get("/fruits") // 1
        .param("search", "ban") // 2
);
```

The class `MockMvcRequestBuilders` provides static methods, such as `get` or `post`, to simulate HTTP requests on a particular endpoint of the controller (`1`). We can add HTTP parameters to the request fluently (`2`).

Furthermore, the `MockMvcRequestBuilders` class provides other static methods:

- HTTP verbs like `delete`, `put` or `patch`.
- `fileUpload` to upload binary files.
- Others ([documentation](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/test/web/servlet/request/MockMvcRequestBuilders.html)).


### Assert

MockMvc permits to add assertions to the controller's response.

```java
mockMvc.perform(get("/fruits")
    .param("search", "ban"))
    .andExpect(status().isOk()) // 1
    .andExpect(view().name("fruits")) // 2
    .andExpect(model().attribute("fruitBowl", contains("banana"))); // 3
```

Assertions are done with the `andExpect()` method. And, the class `MockMvcResultMatchers` provides static methods to do assertions on the HTTP status (`1`), the view asked by the controller (`2`) and the model completed by the controller (`3`).

## Conclusion

The Spring test framework is a powerful, complete and simple way to test Spring MVC controllers. Its fluent API allows writing elegant and yet precise tests.
