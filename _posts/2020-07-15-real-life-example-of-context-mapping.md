---
layout: post
title: Real life example of context mapping
---

I am challenged as a solution architect in my current project to deal with very complex business domain. In order to make our design manageable we apply Domain Driven Design (DDD) a lot. We still in the process of cutting the problem space into domains. However our core domain includes trade processing. One of generic domains is CRM.

The business requirement is to stop Hargreaves Lansdown employee from buying theirs employer shares. Its implementation requires integration between two domains mentioned above. In terms of DDD it is context mapping. There are many model of context mapping like shared kernel, customer-supplier just to name few. We don't want to spoil our trading platform context language with the concept of the employee. On the other hand we have So the obvious choice was to implement context mapping as an anti-corruption layer (ACL). 

![ACL between CRM and Trading]({{ site.baseurl }}/images/ACLbetweenCRMandTrading.png)

A service within CRM bounded context emits ClientUpdated event. This one is sent to a topic named after the event.  ACL within trading is subscribed to the topic.  On every event received it performs translation to one being part of trading's ubiquitous language.  It simple operation of extracting client identifier and checking if the client is an employee. This information is used to build a new command, that is DisableTradingForStock with related client id and hard coded stock ISIN. Command is then sent via message broker to service managing stock allowance for investors.

For example a _ClientUpdated_ event with following body
{% highlight json %}
{"clientId":"1135921",
  .... ,
  "isEmployee": true
} 
{% endhighlight %}
will be transformed into a DisableTradingForStock command with body:
{% highlight json %}
{"clientId":"1135921",
  .... ,
  "stock": {
  "isin": "GB00B1VZ0M25"
}
{% endhighlight %}
