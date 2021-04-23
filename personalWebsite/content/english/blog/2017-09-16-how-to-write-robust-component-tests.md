---
type: "post"
title: "How to Write Robust Component Tests"
date: 2017-09-16 13:37:12 +0200
description: ""
author: Nadia Humbert-Labeaumaz
image : "images/blog/robust-component-tests/header.jpg"
bg_image: "images/blog/robust-component-tests/header.jpg"
categories: ["Software"]
tags: ["software-testing", "software-design"]
draft: false
comments: true
---

Component tests allow to test complete use cases from end to end. They are often expensive especially in terms of setup and execution time. Thus, thought needs to be given to define their scope. Nevertheless, they are required in order to check and document the overall behaviour of the application or the microservice.

I have noticed that, in the context of microservices, these tests are very cost-effective. Indeed, they can be quite easy to set up because the already existing external API of the microservice can often be used directly without needing additional elements (like a fake server for instance). Moreover, the scope of a microservice is generally limited and can be tested exhaustively in isolation.

The goal of this article is to show how to make these tests robust. The main idea is to make them independent of the implementation.

<!-- more -->

The following example shows a Gherkin specification, for a booking HTTP API, that is very coupled to the technical implementation:
```gherkin
Scenario: Get an error when trying to book a hotel with no vacancy
    Given Get to hotel service "/api/hotel/1234" returns a response with the status code 200 and the body:
        """
        {
          "id": "1234",
          "name": "Ritz",
          "availableRooms": 0
        }
        """
    Given Get to user service "/api/user/456" returns a response with the status code 200 and the body:
        """
        {
          "id": "456",
          "firstName": "John",
          "lastName": "Doe"
        }
        """
    When I post "http://my.app.fr:8080/booking/api/" on the "booking" application with the following body:
        """
        {
          "userId": "456",
          "hotelId": "1234",
          "from": "2017-09-16",
          "to": "2017-09-24"
        }
        """
    Then I get a response with the status code 200
    And I get a JSON response with the body:
        """
        {
          "errors" : [ {
            "code" : 12,
            "message" : "There is no room available for this booking request"
          } ]
        }
        """
```

This is the associated Java code to the first step (the framework used is Cucumber):
```java
@Given("^Get to hotel service \"([^\"]*)\" returns a response with the status code (\\d+) and the body:$$")
public void getToHotelServiceReturnsAResponseWithTheStatusCodeAndTheBody(String uri, int statusCode, String body) {
    hotelWireMockServer.stubFor(get(urlEqualTo(uri))
        .willReturn(aResponse()
            .withStatus(statusCode)
            .withHeader("Content-Type", MediaType.APPLICATION_JSON_VALUE)
            .withBody(body)
        )
    );
}
```

It is possible to make the test more explicit and functional. For instance, instead of describing HTTP calls and responses in the steps, it is possible to write them in plain English. The first steps in the Gherkin file can be replaced by:

```gherkin
Given There is no vacancy for the hotel "Ritz" of id 1234
Given The following users exist:
    | id   | firstName | lastName |
    | 456  | John      | Doe      |
```

The associated Java code is now the following:
```java
@Given("^There is no vacancy for the hotel \"([^\"]*)\" of id (\\d+)$")
public void thereIsNoVacancyForTheHotelOfId(String name, Long id) {
    hotelWireMockServer.stubFor(get(urlEqualTo("/api/hotel/" + id))
        .willReturn(aResponse()
            .withStatus(statusCode)
            .withHeader("Content-Type", MediaType.APPLICATION_JSON_VALUE)
            .withBody("{ \"id\": \"" + id + "\", \"name\": \"" + name +"\", \"availableRooms\": 0 }")
        )
    );
}

@Given("^The following users exist:$")
public void theFollowingUsersExist(List<UserDTO> users) {
    users.forEach(
        user -> userWireMockServer.stubFor(get(urlEqualTo("/api/user/" + user.id))
            .willReturn(aResponse()
                .withStatus(200)
                .withHeader("Content-Type", MediaType.APPLICATION_JSON_VALUE)
                .withBody(user.asJson())
            )
        )
    );
}
```

We notice that the purely technical details like the URL, the JSON response and the HTTP status are now specified in the Java code. This allows to make the Gherkin specification more focused on the behaviour, clearer and more concise. Hence, this test is now more maintainable and robust.

The initial test is now the following:
```gherkin
Scenario: Get an error when trying to book a hotel with no vacancy
    Given There is no vacancy for the hotel "Ritz" of id 1234
    Given The following users exist:
        | id   | firstName | lastName |
        | 456  | John      | Doe      |
    When I post "http://my.app.fr:8080/booking/api/" on the "booking" application with the following body:
        """
        {
          "userId": "456",
          "hotelId": "1234",
          "from": "2017-09-16",
          "to": "2017-09-24"
        }
        """
    Then I get a response with the status code 200
    And I get a JSON response with the body:
        """
        {
          "errors" : [ {
            "code" : 12,
            "message" : "There is no room available for this booking request"
          } ]
        }
        """
```

Since this microservice is an HTTP API, it can be relevant to keep the `When` and `Then` in a technical form. Indeed, one can argue that the HTTP status and the format of the exchanged messages are part of its behaviour.

## Conclusion
A component test must explicitely describe a real use case. In order to do that, it is important to make it as independent as possible of the implementation. This article shows a way to go from a test highly coupled to the implementation to a test more functional and concise.
