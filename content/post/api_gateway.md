---
title: "API Gateway & Back-end for Front-end for Microservices"
date: "2018-12-28"
description: ""
tags: 
    - "spring cloud"
    - "backend for frontend"
    - "api gateway"
    - "microservices"
categories:
    - "microservices"
    - "spring-cloud"
    - "api gateway"
highlight: "true"
draft: "true"
---

## Why
Edge Service - API Gateway

Zuul is integrated with the Ribbon load balancer, Hystrix circuit breaker, and service discovery, for example with Eureka. 
By default, Zuul doesn't integrate with Service Discovery. After integrating with Service Discovery, below configuration can be used.
zuul:
 routes:
  account-service:
   path: /account/**
  customer-service:
   path: /customer/**
  order-service:
   path: /order/**
  product-service:
   path: /product/**

Spring Cloud Zuul by default exposes all the services registered in Eureka server.