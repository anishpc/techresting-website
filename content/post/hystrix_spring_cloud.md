---
title: "Netflix Hystrix with Spring Cloud"
date: "2018-11-11"
description: ""
tags: 
    - "spring cloud"
    - "hystrix"
    - "microservices"
categories:
    - "microservices"
    - "spring-cloud"
highlight: "true"
---

## Circuit Breaker

Michael Nygard in this book "Release It" talks about the "Circuit Breaker" stability pattern which works very much inline with electrical circuit breakers which open the circuit on excess usage. Once the usage is back to normal, the circuit should close on its own.
The Netflix Hystrix library has been built on this idea of circuit breaker for working with distributed systems.

***

### Why Now

In a single application (monolith or otherwise), failures can be caught as exceptions and handled accordingly. However, with the wide adoption of microservices, there are a different set of failures which need to be handled for the system to be resilient.
If we have to go by the book ‘Release It’, Hystrix adopts the following Stability patterns:

* Circuit Breaker
* Timeouts (slow responses from downstream applications)
* Bulkheads 
* Fail fast

Even a monolith application would have a number of dependencies which would be accessible over the network and any one of them could potentially fail. However, in the world of 100s of microservices, critical microservices have about a dozen dependencies in the entire call stack. With such a higher number of distributed dependencies, there is high chance that eventually there is an outage in one of the dependent systems or those dependent systems are running in a degraded fashion.
Resilient systems are one which isolate the failure and prevent errors in one system from cascading to upstream systems - the common example being pool exhaustion (thread pools/connection pools).

## Hystrix

Hystrix was designed to solve these types of failures and provides a dashboard for near-realtime monitoring for operations
Read this (https://github.com/Netflix/Hystrix/wiki/FAQ%20:%20General#where-does-the-name-come-from) to know where the name comes from.

### Hystrix & Spring Cloud

Spring Cloud’s annotation based approach provides a low-barrier entry to the Netflix Hystrix library, which internally has much complexity built into it. 

### Scenario

Let's see a scenario in which we display the balance to a user. For this use case, the balance gateway service needs to interact with 3 downstream services :

* Payment Search service
* Billing Gateway 
* Loyalty Points service

Here, the diagram below shows the scenario of the Loyalty Points service being not responsive & how Hystrix can help with that scenario.


![Balance](/img/hystrix_app.png)

### Fallback

Hystrix allows an option in which we can provide an alternate route should the original route not respond. So, in the above case, if the Loyalty Points service does not respond then we can choose to deliver the values from a local cache.
#### The user experience in case of a fallback flow should be discussed with stakeholders. In this case of Loyalty Points being delivered from the local cache, there has to be business logic if that is even allowed. In this case, you have to choose between - "not displaying loyalty points" and "displaying possibly stale values of loyalty points". If it is fine to display stale values (stale within a threshold) then this fallback option should be considered.

### Show me the code

1.	*Build Dependency :*  add the following dependency to your pom xml (or accordingly to gradle build file)
{{< highlight xml >}}
<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>  
</dependency>  
{{< / highlight >}}
       
2. *Configure in Spring Cloud :*	Add `@EnableCircuitBreaker` annotation to the `SpringApplication` class
{{< highlight java "hl_lines=4" >}}
import org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker;
// other code

@EnableCircuitBreaker
@SpringBootApplication
//other annotations
public class GatewayApplication {

    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }
}
{{< / highlight >}}
3.	*Client code :* Wrap the method which makes the outbound call with `@HystrixCommand` annotation
  * fallback method must be in the same class as the wrapped method
  * fallback method must have the same signature of the wrapped method
{{< highlight java "hl_lines=4" >}}
import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
// other code

@HystrixCommand(fallbackMethod = "buildLoyaltyPointsFromCache")
public LoyaltyPointsResponse retrieveLoyaltyPoints(String serviceId) {
        // code to invoke the Loyalty Points service
}

public LoyaltyPointsResponse buildLoyaltyPointsFromCache(String serviceId) {
      // code to retrieve from local cache
}
{{< / highlight >}}


**References : Standing on the shoulders of giants**

* [Book - Release It (by Michael Nygard)](https://pragprog.com/book/mnee2/release-it-second-edition)  (highly recommended)
* [Netflix Hystrix wiki](https://github.com/Netflix/Hystrix/wiki)
* [Martin Fowler on Circuit Breakers](https://martinfowler.com/bliki/CircuitBreaker.html)
* [Book - Spring Microservices in Action (by John Carnell)](https://www.manning.com/books/spring-microservices-in-action)




