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
---

## Why

During our migration from Monolith to microservices, we wanted to split the 'Customer Self-Serve' application. As with all monoliths, this application started out as a service to retrieve customer information for 'Customer Service' applications. However, with each new touch point channel added to serve the customer, the logic for all the channels has been incorporated into that app. The other issue is that this app had become a bottleneck of sorts for changes specific to the channel. Each channel wanted to provide different notifications & messages to the customer, however, all the changes had to be streamlined through the 'Customer Self-serve' release cycles increasing the delivery timelines.
Below is the architecture we had before migrating to microservices.
![Balance](/img/bff_before.png)

## How
After splitting out the functionality from a monolith to microservices, we wanted to create an interface which was specific to the business domain but also catered to each with relevant context-specific information.

Phil Calçado/Sam Newman have talked about the use of Back-end for Front-end architectural pattern which he had used at SoundCloud. This architectural pattern used at the edge fit our use-case.
We created a generic Balance operation API which provided detailed information about the balance & other related information. This service encapsulated all domain-specific information to capture a customer's balance information.

Below is the diagram which we reached after splitting the services based on Backend-For-Frontend pattern.
![Balance](/img/bff_after.png)

## Points to consider 
#### Team 
The client-specific microservice would be maintained & developed by the client team. This leads to each team iterating at their own pace and interacting with a generic single-responsibility API
#### Duplication & Divergence
There is a possibility of duplication amongst the BFF services. [Lukasz Plotnicki](https://www.thoughtworks.com/insights/blog/bff-soundcloud) suggests the following : *"A guiding rule could be: use a shared library if the functionality being extracted doesn't have to be updated at the same time, but if it does, then use a service."* 
#### Domain-Logic Creep
Evolution of BFF services could lead to business-logic creep - where the core business-domain logic which should have been in the generic service or a separate service/library is now spread across channel-specific BFF services which can cause functionality to differ across channels. If there is a functionality which is not specific to that channel then there has to be co-ordination with the generic team and other channels on the implementation approach.



**References : Standing on the shoulders of giants**

* [Sam Newman](https://samnewman.io/patterns/architectural/bff/)
* [Phil Calçado @ SoundCloud](http://philcalcado.com/2015/09/18/the_back_end_for_front_end_pattern_bff.html)
* [Lukasz Plotnicki @ ThoughtWorks](https://www.thoughtworks.com/insights/blog/bff-soundcloud)


