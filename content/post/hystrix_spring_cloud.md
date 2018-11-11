+++
title = "Netflix Hystrix with Spring Cloud"
description = ""
tags = [
    "spring cloud",
    "hystrix",
    "microservices"
]
date = "2018-11-11"
categories = [
    "microservices",
    "spring-cloud"
]
highlight = "true"
+++

## Circuit Breaker

Michael Nygard in this book "Release It" talks about the "Circuit Breaker" stability pattern which works very much inline with electrical circuit breakers which open the circuit on excess usage. Once the usage is back to normal, the circuit should close on its own.
The Netflix Hystrix library has been built on this idea of circuit breaker for working with distributed systems.

***

## Why Now

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

## Hystrix & Spring Cloud

Spring Cloud’s annotation based approach provides a low-barrier entry to the Netflix Hystrix library, which internally has much complexity built into it. 

## Show me the code

Dependency:

1.	To get started : add the following dependency to your pom xml (or accordingly to gradle build file)
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```        
2.	Add `@EnableCircuitBreaker` to the `SpringApplication` class
3.	Wrap the method which makes the outbound call with `@HystrixCommand` annotation


Standing on the shoulders of giants :

