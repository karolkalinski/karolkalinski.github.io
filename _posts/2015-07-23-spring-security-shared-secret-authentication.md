---
layout: post
title: Spring Security shared secret authentication for microservices 
---

So we all are building microservices, now. Most of them provides RESTful APIs that enables other ones to perform actions. One day we are realizing that allowing everybody to access an endpoint may lead to security breach. In this article I will show how we can quickly add a shared secret based security to an application, that is based on [micro-infra-spring](https://github.com/4finance/micro-infra-spring "micro infra spring"), a project with aim to speed up a microservice creation using spring secuirty authentication filters.        

## The problem

In our enterprise application we have two microservices that cooperate together to perform a business task. One of the microservices is working outside of the context of the user logged into our enterprise application, e.g. is an adapter responsible for pulling data from an external system. The other needs to provide an interface that allows posting significant data to fulfill requirements of the first one. Therefore we are going to create a session-less RESTful endpoint, that is secured against unrestricted access. 

## The easiest solution
The easiest solution I have encoutured few times is to allow access only from selected hosts by providing thiers ip adresses in the endpoint security configuration. In order to enable such way of protection in micro-infra-spring based microservice all we have to do is to add [a spring security configuration](http://mvnrepository.com/artifact/org.springframework.security/spring-security-config/4.0.2.RELEASE) and [a spring secuirty web](http://mvnrepository.com/artifact/org.springframework.security/spring-security-web/4.0.2.RELEASE) to its dependencies and adapt the default Spring Boot security configuration. More info on adding spring security to Spring Based web application can be found here: [Securing a Web Application](https://spring.io/guides/gs/securing-web/ "Securing a Web Application"). Adapting the configuration is even easier:
{% highlight java %}
```
@Configuration
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                .antMatchers("/secret")
                .hasIpAddress("::1")
                .and()
                .csrf()
                .disable();
    }
}
```
{% endhighlight %}
The code above tells spring security to allow access to all urls that matcher patern `/secret` from the local host (`::1` is ipv6 address for loopback). As far as we are going to create a session-less endpoint we should disable the CSFR, that is enabled by default for spring security set up by Java configuration. We achieve it by chaning `csfr` and `disable`. This solution has one huge disadvantage. Its not scalable at all.

##Plain text shared secret

## Advantages
The authentication is performed in a single request without any redirection
No artificial user is required for authentication for an endpoint

If you want to be super-paranoid safe, see: org.springframework.security.crypto.bcrypt.BCrypt#checkpw and https://en.wikipedia.org/wiki/Timing_attack
