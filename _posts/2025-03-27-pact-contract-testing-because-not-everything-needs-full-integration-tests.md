---
layout: post
title: "PACT Contract Testing - Because Not Everything Needs Full Integration Tests"
date: 2025-03-27 16:30:00 +0000
categories: Testing
tags: testing pact contract-testing integration-testing java spring-boot
---

# PACT Contract Testing - Because Not Everything Needs Full Integration Tests

I am a big fan of integration tests. I think they are the best way to test that your system works as a whole, but
sometimes, integration tests are overkill or too complex to set up that the return on investment is just not worth it.
So what do you do then? You could write unit tests, but they only test the applications in isolation. That's where
contract testing comes in. 

Looking at the table below, you can see that contract tests are a good middle ground between unit and integration tests. 
They are fast to run, easy to set up, and provide a good level of coverage.

| Test Type   | Complexity | Speed | Coverage |
| ----------- | ---------- | ----- | -------- |
| Unit        | Low        | Fast  | Low      |
| Contract    | Low        | Fast  | Medium   |
| Integration | High       | Slow  | High     |

In this post, I will explain what contract testing is, why it is useful, and how to use PACT to write contract tests for
your applications. 

## What is contract testing?

Contract testing is a way to test the interactions between two applications. It allows you to define a contract between 
the two applications, which specifies how they should interact. This contract can then be used to test both systems in 
isolation, without the need for a full integration test.

## What is PACT?

[PACT](https://docs.pact.io/) is a contract testing tool that allows you to define a contract between two applications. 
It is a consumer-driven contract testing tool, which means that the consumer of the API defines the contract. The 
provider of the API then uses this contract to test that it meets the requirements of the consumer. This allows you to 
test the interactions between the two applications without the need for a full integration test. 

PACT is available in multiple languages, including Java, JavaScript, Ruby, and Go. This means that you can use PACT to 
test applications written in different languages, which is especially useful in microservices architectures where 
different services may be written in different languages. However, I have found that the PACT libraries for each language 
do not have feature parity. For example the Java version of PACT is much more mature than the Python version. So, ensure 
you verify the level of support before committing to using PACT in a less-supported language. You can find the list of 
supported languages [here](https://docs.pact.io/getting_started/specification#client-language-support).

PACT is not just limited to REST APIs. It also supports testing interactions with message brokers, such as Kafka.

## How does PACT work?

PACT works by defining a contract between the consumer and provider of an API. The consumer defines the contract by
writing a unit test that specifies the expected interactions with the API. This test is then used to generate a contract 
file, which is a JSON file that contains the details of the contract. The provider then uses this contract file to test 
that it meets the requirements of the consumer.

### The contract file

You can see an example of a contract file below:

```json
{
  "consumer": {
    "name": "OrderConsumer"
  },
  "provider": {
    "name": "OrderProvider"
  },
  "interactions": [
    {
      "description": "a request for a order",
      "providerStates": [
        {
          "name": "order with ID 88 exists"
        }
      ],
      "request": {
        "method": "GET",
        "path": "/order/88",
        "headers": {
          "Accept": "application/json"
        }
      },
      "response": {
        "status": 200,
        "headers": {
          "Content-Type": "application/json"
        },
        "body": {
          "id": 1,
          "items": [
            "skateboard",
            "delorean"
          ],
          "date": "2025-03-26T12:00:00Z"
        }
      },
      "matchingRules": {
        "body": {
          "$.id": {
            "combine": "AND",
            "matchers": [
              {
                "match": "type"
              }
            ]
          },
          "$.items": {
            "combine": "AND",
            "matchers": [
              {
                "match": "type"
              }
            ]
          },
          "$.date": {
            "combine": "AND",
            "matchers": [
              {
                "format": "yyyy-MM-dd'T'HH:mm:ss",
                "match": "date"
              }
            ]
          }
        }
      }
    }
  ]
}
```

The contract file contains the details of the interactions between the consumer and provider. It specifies the request
and response details, including the HTTP method, path, headers, and body. It also specifies the matching rules for the
request and response bodies, which allows you to specify things such as date patterns. The matching rules specify how 
the request and response bodies should be matched, which allows you to test that the provider meets the requirements of 
the consumer. It also specifies the provider state, which can be used to set up the state of the provider before the 
interaction is verified, for example creating a mock order with ID 88 in your database or in-memory store. This will 
ensure that when the consumer sends a request for order ID 88, it will receive a valid response.

You can find a example of a contract file [here](https://github.com/adamdougal/pact-testing/blob/main/pacts/PactDemoConsumer-PactDemoProvider.json).


### The consumer side

The consumer side of PACT is where you define the contract. The first step is to create a test class for your client
that will call the provider system:

```java
package com.example.pactdemo.client;

import au.com.dius.pact.consumer.MockServer;
import au.com.dius.pact.consumer.dsl.LambdaDslJsonBody;
import au.com.dius.pact.consumer.dsl.PactDslWithProvider;
import au.com.dius.pact.consumer.junit5.PactConsumerTest;
import au.com.dius.pact.consumer.junit5.PactTestFor;
import au.com.dius.pact.core.model.V4Pact;
import au.com.dius.pact.core.model.annotations.Pact;
import au.com.dius.pact.core.model.annotations.PactDirectory;

// full import list omitted

@PactConsumerTest
@PactDirectory("../pacts")
@PactTestFor(providerName = "PactDemoProvider")
public class OrderClientTest {
```

You then define the contract using the `@Pact` annotation. The `PactDslWithProvider` class is used to define and build 
the request and response details, including the HTTP method, path, headers, and body. The `LambdaDslJsonBody` class is 
used to define the matching rules for the request and response bodies:

```java
    @Pact(provider = "PactDemoProvider", consumer = "PactDemoConsumer")
    public V4Pact orderPact(PactDslWithProvider builder) {
          LambdaDslJsonBody expectedResponseBody = newJsonBody((expectedOrder) -> {
                expectedOrder.stringType("id", "88");
                expectedOrder.object("user", (user) -> {
                      user.stringType("id", "881985");
                      user.stringType("firstName", "Marty");
                      user.stringType("lastName", "McFly");
                      user.stringType("email", "McFlyinTime@futurebound.com");
                });
                expectedOrder.array("items", (item) -> {
                      item.object((itemObj) -> {
                            itemObj.stringType("id", "1");
                            itemObj.stringType("name", "Hoverboard Revamp Kit");
                            itemObj.stringType("description",
                                        "Turn your trusty hoverboard into a sleek, futuristic ride. This upgrade kit comes complete with LED accents and aerodynamic enhancements—perfect for zipping through time and space.");
                            itemObj.numberType("price", 159.99);
                      });
                      item.object((itemObj) -> {
                            itemObj.stringType("id", "2");
                            itemObj.stringType("name", "Time-Traveling Sneaker Laces");
                            itemObj.stringType("description",
                                        "Infuse your kicks with retro-futuristic flair! These self-adjusting, luminous sneaker laces are crafted for high-speed adventures, ensuring your style is always ahead of its time.");
                            itemObj.numberType("price", 24.99);
                      });
                });
                expectedOrder.numberType("totalPrice", 184.98);
                expectedOrder.date("orderDate", "yyyy-MM-dd'T'HH:mm:ss", Date.from(LocalDateTime.parse("2015-10-21T10:30:00").toInstant(UTC))); 
          });

          return builder
                      .given("order with ID 88 exists")
                      .uponReceiving("a request to get an order by id")
                      .path("/order/88")
                      .method("GET")
                      .headers("Accept", "application/json")
                      .willRespondWith()
                      .status(200)
                      .headers(Map.of("Content-Type", "application/json"))
                      .body(expectedResponseBody.build())
                      .toPact(V4Pact.class);
    }
```

Finally, you can use the `@PactTestFor` annotation to specify the provider system that you are testing against:

```java
    @Test
    @PactTestFor(pactMethod = "orderPact")
    void testGetOrder(MockServer mockServer) throws Exception {
          OrderClient orderClient = new OrderClient(mockServer.getUrl());

          Order order = orderClient.getOrder("88");

          assertNotNull(order);
    }
```

The `@PactTestFor` annotation specifies the provider system that you are testing against, `orderPact` in this example.
This needs to match the name of the method that defines the contract.

Pact starts a mock server that simulates the provider system. The `MockServer` object is used to get the URL of the mock
server, which is then passed to the `OrderClient` to test against.

See [here](https://github.com/adamdougal/pact-testing/blob/main/consumer/src/test/java/com/example/pactdemo/client/OrderClientTest.java)
for a full working example of the consumer side of PACT.

### The provider side

The provider side of PACT is where you verify that the provider system meets the requirements of the consumer. The
first step is to create a test class for your provider system that will verify the contract. In our example, the 
provider system is a Spring Boot application, so we can use the `@SpringBootTest` annotation to start the application
and run the tests against it:

```java
package com.example.pactdemo;

import au.com.dius.pact.provider.junit5.HttpTestTarget;
import au.com.dius.pact.provider.junit5.PactVerificationContext;
import au.com.dius.pact.provider.junit5.PactVerificationInvocationContextProvider;
import au.com.dius.pact.provider.junitsupport.Provider;
import au.com.dius.pact.provider.junitsupport.State;
import au.com.dius.pact.provider.junitsupport.loader.PactFolder;

// full import list omitted

@Provider("PactDemoProvider")
@PactFolder("../pacts")
@ExtendWith(SpringExtension.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class PactDemoApplicationTests {
```

We then need to setup the test context and specify the target URL of the provider system. As we're starting up the 
Spring Boot application, we can use the `@LocalServerPort` annotation to get the port that the application is running on
and then set the target URL in the `setUp` method. However, you can also test against a real deployed service if you
wish to be setting the target URL to the real service:

```java
    @LocalServerPort
	  int port;

    @BeforeEach
    void setUp(PactVerificationContext context) {
          context.setTarget(new HttpTestTarget("localhost", port, "/"));
    }

    @TestTemplate
    @ExtendWith(PactVerificationInvocationContextProvider.class)
    void verifyPact(PactVerificationContext context) {
      context.verifyInteraction();
    }
```

The `@TestTemplate` verifies the interactions defined in the contract file. You can also use this to add other things
such as authentication or other headers that are required by the provider system.

We now need to define our tests:

```java
	@State({
		"order with ID 88 exists",
		"order with ID 999 does not exist"
	})
	void testPactWhenOrderWithIdRequestReceived() {
		// This method is used to set up the state of the provider before the interaction is verified.
		// You can use this method to create any necessary data or perform any actions needed to set up the state.
		// For example, you could create a mock order with ID 88 in your database or in-memory store.
		// This will ensure that when the consumer sends a request for order ID 88, it will receive a valid response.
		// Similarly, you can set up the state for order ID 999 to return a 404 response.
	}
```

The `@State` annotation specifies the provider state that is required for the interaction to be verified. The rest of
the test is empty, as the `@TestTemplate` will take care of verifying the interaction. But if you need to set up any
data or perform any actions needed to set up the state, you can do that here.

See [here](https://github.com/adamdougal/pact-testing/blob/main/provider/src/test/java/com/example/pactdemo/PactDemoApplicationTests.java)
for a full working example of the provider side of PACT.

### Where to store the contract files

In the examples above, we have used the `@PactDirectory` annotation to specify the location of the contract files. These
can then just be committed to git. However, unless you're using a monorepo, you will need to store the contract files in
a shared location that is accessible to both the consumer and provider systems. This could be a dedicated contract file
repostitory, where the consumers will need to manually copy the contract files and the providers will need to test
against a pre-deployed system.

Alternatively, you can use the [PACT broker](https://docs.pact.io/pact_broker) to store the contract files. The PACT 
broker is a service that stores the contracts. A [Docker image](https://docs.pact.io/pact_broker/docker_images) is 
available for the PACT broker, as well as [Helm charts](https://docs.pact.io/pact_broker/kubernetes/readme) for 
deployment to Kubernetes.

## Conclusion

So to summarise, PACT is a fantastic contract testing tool that takes the pain out of hand crafting contracts. If
integration tests are overkill or too complex to set up, then PACT is a great alternative.

You can find a full end-to-end example of PACT in action in the repository
[https://github.com/adamdougal/pact-testing/](https://github.com/adamdougal/pact-testing/).
