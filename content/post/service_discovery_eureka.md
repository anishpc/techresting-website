---
title: "Service Discovery with Netflix Eureka (with Netflix Ribbon)"
date: "2018-12-11"
description: ""
tags: 
    - "spring cloud"
    - "eureka"
    - "ribbon"
    - "microservices"
categories:
    - "microservices"
    - "spring-cloud"
highlight: "true"
---

*** Service Discovery

## Why/How

In traditional (prior microservices) systems, there would be a fixed number of service instances. The consuming applications would configure the host-port information of those individual services or of a load balancer in front of those service instances. This worked fine when there was a static & fixed view of the infrastructure. 

However, services designed to be cloud-native can fail,scale in and out. Also, microservices are increasingly being deployed in a containerized environment. In such a dynamic environment, a fixed set of host-port information doesn't scale.

*** Types of Service Discovery:
**** Client-side Service Discovery: 
This has the common approach of configuring the host-port details of the services in the consuming applications. The client uses that host-port details to invoke the service, typically in a round-robin fashion. In this case, there are no health checks for the downstream services and its upto the consuming client to implement some functionality if the client is not able to reach a particular service instance

**** Load Balancer-Based Service Discovery:
In this pattern, the consuming applications configure a load-balancer DNS name, typically, in the application property files. Here, the load balancer would perform the routing. Also, the load balancer can have health checks and other level-7 smarts. In AWS, the Application Load Balancer (ALB) provides routing based on different set of rules like zones, content-based amongst others. The consuming application does not need to know about the routing logic or any load-balancing.

**** DNS Based Service Discovery:
DNS can have different routing policies like latency, geolocation. For the AWS DNS solution -- Route53, here is the entire list of routing policies. https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html. DNS based discovery differ in how the values expire and in turn, how the values are provided to the applications trying to retrieve the service information.

**** Registry based Service Discovery:
In this case, the consuming application queries the Service Registry with some key (typically the service name) which uniquely identifies the service to be invoked. The Service Registry then provides the details of the service to the consuming application. 
The services register with the Service Registry when they come online and then send regular heartbeat messages to the registry server.
The Service Discovery server has the smarts to identify the live instances of the service and filtering based on other conditions (like zones, etc.)
The Service Discovery solution provided by Netflix for this pattern is Eureka. Other alternatives are Consul, Zookeeper, etc.
 

***