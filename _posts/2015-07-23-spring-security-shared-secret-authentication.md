---
layout: post
title: Spring Security shared secret authentication for microservices 
---

So we all are building microservices, now. Most of them provides rest APIs that enables other ones to perform actions. One day we are realizing that allowing everybody to access an endpoint may lead to security breach. In this article I will show how we can quickly add a shared secret based security to an application, that is based on [micro-infra-spring](https://github.com/4finance/micro-infra-spring "micro infra spring"), a project with aim to speed up a microservice creation using spring secuirty authentication filters.        

## The problem

In our enterprise application we have two microservices that cooperate together to perform a business task. One of the microservices is working outside of the context of the user logged into our enterprise application, e.g. is an adapter responsible for pulling data from an external system. The other needs to provide an interface that allows posting significant data to fulfill requirements of the first one. Therefore we are going to create an endpoint, that is protected against unrestricted access. 

## The easiest solution
The easiest solution I have encoutured few times is to allow access only from selected hosts by providing thiers ip adresses in the endpoint security configuration. In order to enable such way of protection in micro-infra-spring based microservice all we have to do is to add a spring security on its classpath and adapt the default Spring Boot security configuration. More info on adding spring security to Spring Based web application can be found here: [Securing a Web Application](https://spring.io/guides/gs/securing-web/ "Securing a Web Application"). Adapting the configuration is even easier:

If you want to be super-paranoid safe, see: org.springframework.security.crypto.bcrypt.BCrypt#checkpw and https://en.wikipedia.org/wiki/Timing_attack
