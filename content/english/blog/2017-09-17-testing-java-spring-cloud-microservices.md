---
type: "post"
title: "Testing Java Spring Boot Microservices"
slug: "testing-java-spring-cloud-microservices"
date: 2017-09-17 16:09:24 +0200
description: ""
author: Nadia Humbert-Labeaumaz
image : "images/blog/testing-microservices/header.jpg"
bg_image: "images/blog/testing-microservices/header.jpg"
categories: ["Software"]
tags: ["Software Testing", "Software Design", "Java"]
draft: false
comments: true
---

Tests are an essential part of our codebase. At the very least, they minimize the risk of regression when we modify our code. There are several types of tests, and each has a specific role: unit tests, integration tests, component tests, contract tests and end-to-end tests. Therefore, it is crucial to understand the role of each type of test to leverage its potential.

The goal of this article is to describe a strategy to use them in order to test Java Spring Boot microservices. For every type of test, we will explain its role, scope, and the tooling we like to use.

<!--more-->

## Anatomy of a Microservice

First, we will set up a common vocabulary to make this article as clear as possible.

A standard microservice is composed of:

- **Resources**: HTTP controllers or AMQP listeners that will serve as the entry point of the microservice.
- **Services/domain**: Classes that will contain the business logic.
- **Repositories**: Classes that will expose an API to access storage (like a database, for instance).
- **Clients**: HTTP clients or AMQP producers that will communicate with external resources.
- **Gateways**: Classes that will serve as interfaces between domain services and clients by handling HTTP or AMQP related tasks and providing a clean API to the domain.

## Types of Tests

### Unit Tests

Unit tests allow testing a unit (generally a method) in isolation and are very cost-effective: easy to set up and very fast. As a result, they can give fast feedback about the application's state to spot bugs or regressions. It is then advised to test every edge case and relevant combination with unit tests.
As a bonus, they help validate a design: if the code is difficult to test, the design is probably bad.

In a microservice, like in any other codebase, it is essential to unit test domain/service classes and every class containing logic.

The tooling we prefer to write unit tests is [Junit](http://junit.org/junit5/) (to run the tests), [AssertJ](http://joel-costigliola.github.io/assertj/) (to write assertions) and [Mockito](http://site.mockito.org/) (to mock external dependencies).

### Integration Tests

Integration tests are used to test the proper integration of the different bricks of the application. They are sometimes hard to set up and have to be carefully chosen. The idea is not to test all possible interactions but to choose relevant ones. The feedback of these tests is less fast than with unit tests because they are slower to execute. It is important to note that writing too many integration tests for the same interaction can be counter-productive. Indeed, the build time will be increased without any added value.

In a microservice, it is relevant to write integration tests for:

- **Repositories** -- when the query is more complex than just a `findById`.
- **Services** -- in case of doubt on the interaction between the service and repository (JPA or transaction issues, for instance).
- **HTTP clients**.
- **Gateways**.

Spring Boot provides great tooling to write integration tests. A typical integration test with Spring Boot looks like this:

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class UserGatewayIntTest {

    @Autowired
    private UserGateway userGateway;

    // ...
}
```

The test must use the `SpringRunner` and be annotated with `@SpringBootTest`. It is then possible to inject a bean using `@Autowired` and mock one using `@MockBean`.
Also, the database should be embedded (H2 database is a good candidate) for the integration tests to be executable anywhere. For the same reason, external HTTP resources can be mocked using [WireMock](http://wiremock.org/) and an SMTP server with [SubEthatSMTP](https://github.com/voodoodyne/subethasmtp).

To mock external microservices, it is necessary to fix the port. In production, microservices will register themselves to a registry and dynamically get an URL assigned. In tests, it is possible to fix the URL in tests by adding a property to the test `application.yml` if [Ribbon](https://github.com/Netflix/ribbon) is used with Spring Cloud (here, the external microservice name is `user`):

```yml
user:
  ribbon:
    listOfServers: localhost:9999
```

### Component Tests

Component tests allow testing complete use cases from end to end. However, they are often expensive, especially in terms of setup and execution time. Thus, thought needs to be given to defining their scope. Nevertheless, they are required to check and document the overall behaviour of the application or the microservice.

In the context of microservices, these tests are very cost-effective. Indeed, they can be quite easy to set up because it is often possible to directly use the existing external API of the microservice without needing additional elements (like a fake server, for instance). Moreover, the scope of a microservice is generally limited and can be tested exhaustively in isolation.

Component tests should be concise and easy to understand (see [How to Write Robust Component Tests](https://blog.crafties.fr/2017/09/16/how-to-write-robust-component-tests/)). The goal is to test the behaviour of the microservice by writing a nominal case and very few edge cases. We noticed that writing the specification before implementing the feature can lead to straightforward component tests. Moreover, it is good practice to write the component tests in collaboration with the different project stakeholders to cover the feature effectively.

In component tests, it is possible to use an embedded database. Moreover, it is possible to mock HTTP and AMQP clients: this is not the place to test the integration with external resources (see [Setup a Circuit Breaker with Hystrix, Feign Client and Spring Boot](https://blog.crafties.fr/2017/07/23/setup-a-circuit-breaker-with-hystrix/)).

An example of tooling we can use to write component tests is [Gherkin](https://github.com/cucumber/cucumber/wiki/Gherkin) (to write the specifications) with [Cucumber](https://cucumber.io/) (to run the tests).

To perform requests on the HTTP API of the microservice and make assertions on the response, [MockMvc](https://blog.crafties.fr/2015/10/31/testing-spring-mvc-controllers/) can be used.
```java
@WebAppConfiguration
@SpringBootTest
@ContextConfiguration(classes = AppConfig.class)
public class StepDefs {

    @Autowired private WebApplicationContext context;
    private MockMvc mockMvc;
    private ResultActions actions;

    @Before public void setUp() {
        mockMvc = MockMvcBuilders.webAppContextSetup(context).build();
    }

    @When("^I get \"([^\"]*)\" on the application$")
    public void iGetOnTheApplication(String uri) throws Throwable {
        actions = mockMvc.perform(get(uri));
    }

    @Then("^I get a Response with the status code (\\d+)$")
    public void iGetAResponseWithTheStatusCode(int statusCode) throws Throwable {
        actions.andExpect(status().is(statusCode));
    }
}
```

To inject AMQP messages, the channel used by Spring Cloud Stream can also be injected directly into the test.
```java
// AMQP listener code
@EnableBinding(MyStream.Process.class)
public class MyStream {

    @StreamListener("myChannel")
    public void handleRevision(Message<MyMessageDTO> message) {
      // handle message
    }

    public interface Process {
        @Input("myChannel") SubscribableChannel process();
    }
}

// Cucumber step definition
public class StepDefs {

    @Autowired
    private MyStream.Process myChannel;

    @When("^I publish an event with the following data:$")
    public void iPublishAnEventWithTheFollowingData(String payload) {
        myChannel.process().send(new GenericMessage<>(payload));
    }
}
```

Finally, it may be important to fix the time to make tests more robust (see [Controlling the Time in Java](https://blog.crafties.fr/2017/07/20/controlling-time-in-java/))
```java
public class StepDefs {

    @Autowired @MockBean private ClockProvider clockProvider;

    @Given("^The time is \"([^\"]*)\"$")
    public void theTimeIs(String datetime) throws Throwable {
        ZonedDateTime date = ZonedDateTime.parse(datetime);
        when(clockProvider.get()).thenReturn(Clock.fixed(date.toInstant(), date.getZone()));
    }
}
```

### Contract Tests

The goal of contract tests is to automatically verify that the provider of a service and its consumers speak the same language. These tests do not aim to verify the behaviour of the components but simply their contracts. Moreover, they are handy for microservices since almost all their value lies in their interactions. Indeed, it is crucial to guarantee that no provider breaks the contract used by its consumers.

The general idea is that consumers write tests that define the initial state of the provider, the request sent by the consumer and the expected response. The provider must supply a server in the required state. The contract will automatically be verified against this server. This implies the following:

- **on the consumer side**: contract tests are written using the HTTP client. Given a provider state, assertions are made on the HTTP response.
- **on the provider side**: only the HTTP resource should be instantiated. All its dependencies should be mocked to provide the required state.

It is important to note that contract tests should stick to the real needs of the consumer. If a consumer does not use a field, it should not be tested in the contract test. Then, the provider is free to update or delete every field that is not used by any consumer and we are sure that if tests fail, it is for a good reason.

The tool we like to use to write and execute contract tests is [Pact](https://docs.pact.io/). It is a very mature product that has plugins for a lot of languages (JVM, Ruby, .NET, Javascript, Go, Python, etc.). Moreover, it is well integrated with Spring MVC thanks to the [DiUS pact-jvm-provider-spring plugin](https://github.com/DiUS/pact-jvm/tree/master/pact-jvm-provider-spring).
During the execution of the consumer tests, contracts (called pacts) are generated in JSON format. They can be shared with the provider using a service called the [Pact Broker](https://github.com/pact-foundation/pact_broker).

This is an example of a consumer test written with the [DiUS pact-jvm-consumer-junit plugin](https://github.com/DiUS/pact-jvm/tree/master/pact-jvm-consumer-junit):
```java
@Test
public void should_send_booking_request_and_get_rejection_response() throws AddressException {

    RequestResponsePact pact = ConsumerPactBuilder
            .consumer("front-office")
            .hasPactWith("booking")
            .given("The hotel 1234 has no vacancy")
            .uponReceiving("a request to book a room")
            .path("/api/book")
            .method("POST")
            .body("{" +
                    "\"hotelId\": 1234, " +
                    "\"from\": \"2017-09-01\", " +
                    "\"to\": \"2017-09-16\"" +
            "}")
            .willRespondWith()
            .status(200)
            .body("{ \"errors\" : [ \"There is no room available for this booking request.\" ] }")
            .toPact();

    PactVerificationResult result = runConsumerTest(pact, config, mockServer -> {
        BookingResponse response = bookingClient.send(aBookingRequest());
        assertThat(response.getErrors()).contains("There is no room available for this booking request.");
    });

    assertEquals(PactVerificationResult.Ok.INSTANCE, result);
}
```

On the server side:
```java
@RunWith(RestPactRunner.class)
@Provider("booking")
@Consumer("front-office")
@PactBroker(
        host = "${PACT_BROKER_HOST}", port = "${PACT_BROKER_PORT}", protocol = "${PACT_BROKER_PROTOCOL}",
        authentication = @PactBrokerAuth(username = "${PACT_BROKER_USER}", password = "${PACT_BROKER_PASSWORD}")
)
public class BookingContractTest {

    @Mock private BookingService bookingService;
    @InjectMocks private BookingResource bookingResource;

    @TestTarget public final MockMvcTarget target = new MockMvcTarget();

    @Before
    public void setUp() throws MessagingException, IOException {
        initMocks(this);
        target.setControllers(bookingResource);
    }

    @State("The hotel 1234 has no vacancy")
    public void should_have_no_vacancy() {
        when(bookingService.book(eq(1234L), any(), any())).thenReturn(BookingResult.NO_VACANCY);
    }
}
```

### End-to-End Tests

End-to-end tests need the whole platform to be up and running to run entire business use cases across multiple microservices. Unfortunately, they are costly and slow to run. These tests can be performed manually on a dedicated platform but have to be chosen with great care to maximize their benefits.

## To Sum Up

![Testing strategy](/images/blog/testing-microservices/strategy.png)

## Conclusion

Automatic tests are essential in the software development industry. A good testing strategy can help write more relevant, robust and maintainable tests. This article describes an example of a strategy to test Java Spring Boot microservices.
