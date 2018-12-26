---
title: "Service Discovery with Netflix Eureka (with Netflix Ribbon & Feign)"
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

## Why

In traditional (prior microservices) systems, there would be a fixed number of service instances. The consuming applications would configure the host-port information of those individual services or of a load balancer in front of those service instances. This worked fine when there was a static & fixed view of the infrastructure. 

However, services designed to be cloud-native can fail,scale in and out. Also, microservices are increasingly being deployed in a containerized environment. In such a dynamic environment, a fixed set of host-port information doesn't scale.

## Types of Service Discovery
#### Client-side Service Discovery:
This has the common approach of configuring the host-port details of the services in the consuming applications. The client uses that host-port details to invoke the service, typically in a round-robin fashion. In this case, there are no health checks for the downstream services and its upto the consuming client to implement some functionality if the client is not able to reach a particular service instance

#### Load Balancer-Based Service Discovery
In this pattern, the consuming applications configure a load-balancer DNS name, typically, in the application property files. Here, the load balancer would perform the routing. Also, the load balancer can have health checks and other level-7 smarts. In AWS, the Application Load Balancer (ALB) provides routing based on different set of rules like zones, content-based amongst others. The consuming application does not need to know about the routing logic or any load-balancing.

#### DNS Based Service Discovery
DNS can have different routing policies like latency, geolocation. For the AWS DNS solution -- Route53, here is the entire list of routing policies. https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html. DNS based discovery differ in how the values expire and in turn, how the values are provided to the applications trying to retrieve the service information.

#### Registry based Service Discovery
In this case, the consuming application queries the Service Registry with some key (typically the service name) which uniquely identifies the service to be invoked. The Service Registry then provides the details of the service to the consuming application. 
The services register with the Service Registry when they come online and then send regular heartbeat messages to the registry server.
The Service Discovery server has the smarts to identify the live instances of the service and filtering based on other conditions (like zones, etc.)
The Service Discovery solution provided by Netflix for this pattern is Eureka. Other alternatives are Consul, Zookeeper, etc.
 
## Netflix Eureka

#### Example

![Balance](/img/eureka.png)

Services : 
* In the above example, the services -- 'Payment Search', 'Billing Service' & 'Loyalty Points Svc' would register with the Eureka server when these individual service instances come online. As shown in the image, there could be multiple instances of each of those services. Each service instance would send its own metadata when registering with the Eureka server. The default metadata information include hostname, IP address, port numbers, status page and health check.
* Each of the service instance would send a regular heartbeat message 



#### Using Eureka Client with Ribbon

This is the configuration which would be used in the services which would lookup details of other services which it needs to interact. The information is looked up from Eureka Server.

Ribbon is a client-side load-balancer. Feign uses Ribbon under the hood & with Spring Cloud, Feign provides a load balanced http client.

1.	*Build Dependency :*  add the following dependency to your pom xml (or accordingly to gradle build file)
{{< highlight xml >}}
<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>  
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
</dependency>
{{< / highlight >}}
or select "Eureka Discovery" & "Ribbon" when starting a project from https://start.spring.io/ page.

2. *Configure in Spring Cloud :*	Add `@EnableDiscoveryClient` annotation to the `SpringApplication` class
{{< highlight java >}}
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;
// other code

@EnableDiscoveryClient
@SpringBootApplication
//other annotations
public class GatewayApplication {

    @LoadBalanced
	@Bean
	RestTemplate restTemplate() {
		return new RestTemplate();
	}

    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }
}
{{< / highlight >}}

3. *Invoking other services :*      Use the service names to refer to the service host name `loyalty-points-svc`. The actual host & port details would be used from Eureka
{{< highlight java >}}
public class BalanceGatewayService {
    @Autowired
    private RestTemplate restTemplate;

    public LoyaltyPoints buildLoyaltyPointsBalance(String phoneNumber) {
        LoyaltyPointsDetails loyaltyPoints = restTemplate.getForObject("http://loyalty-points-svc/phone-number/{phoneNumber}", LoyaltyPointsDetails.class, phoneNumber);
        // more code
    }        
}
{{< / highlight >}}


#### Using Eureka Client with Feign Client
1. *Dependency for Feign*
{{< highlight xml >}}
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
{{< / highlight >}}
or select `Feign` from when starting a project from https://start.spring.io/ page.

2. *Client defintition for Feign* - The BalanceGatewayService would be invoking the 'LoyaltyPointsService'. So, in the 'BalanceGatewayService', we need to define the client which is similar to the 'LoyaltyPointsService' interface.
{{< highlight java >}}
@FeignClient(name="loyalty-points-svc",configuration=FeignConfiguration.class)
public interface GatewayDemoClient {
    @GetMapping("/balance")
    Balance getBalance();
}
{{< / highlight >}}

3. *Customizing Feign Configuration* -- Configuration for Feign can be customized in the class provided with the `configuration` attribute for the `@FeignClient` annotation. 
{{< highlight java >}}
@Configuration
public class FeignConfiguration {
    @Bean
    public IRule ribbonRule() {
        return new WeightedResponseTimeRule();  
    }
}
{{< / highlight >}}
`WeightedResponseTimeRule` - Initially, the requests are load balanced amongst the server instances of `loyalty-points-svc`. However, the instances with shorter response times would be given more weightage and over a period of time, the instances with shorter response times would be preferred.

4. *Property file configuration* - In `application.yml`, the properties for Feign can be configured as below:
{{< highlight java >}}
feign:
  client:
    config:
      loyalty-points-svc:
        connectTimeout: 5000
        readTimeout: 5000
{{< / highlight >}}
Use the value of `default` instead of `loyalty-points-svc`, to make this configuration for all Feign clients

**References : Standing on the shoulders of giants**
* [Netflix Eureka Wiki](https://github.com/Netflix/eureka/wiki)
* [Nginx Microservices Guide](https://www.nginx.com/blog/service-discovery-in-a-microservices-architecture/)
* [Spring Cloud Guide](https://cloud.spring.io/spring-cloud-static/spring-cloud.html#_service_discovery_eureka_clients)
