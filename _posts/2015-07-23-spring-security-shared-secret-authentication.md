---
layout: post
title: Spring Security shared secret authentication for microservices 
---

So we all are building microservices, now. Most of them provides rest api's that enables other ones to perform actions. One day we are realising that allowing everybody to access an endpoint may lead to security breach. In this article I will show how we can quickly add a shared secret based security to an application, that is based on [micro infra spring link](https://github.com/4finance/micro-infra-spring "micro infra spring"), a project with aim to speed microservice creation.        

## The problem

We have two microservices that cooperte with each other to perform a business task.  
