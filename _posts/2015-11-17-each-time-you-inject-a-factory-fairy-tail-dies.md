---
layout: post
title: Each time you inject a factory fairy tail dies
---

We are accustomed to using design patterns to solve common problems. Factory is a very popular one. Are you sure, that your architecture is proper, when you inject a factory as a dependency to your component. I will try to prove, that it is not.

## Resposibility for creating objects

The factory design pattern is a very popular design pattern used to hide the dependencies of the objects creation. Each consumer needs to take care about creating the required object on its own. The inversion of control is somehow similar. We are hiding creation details. But we are moving a bit further. We would like to outsource the responsibility of building the objects to external code. So when we are injecting factory then we are mixing this two separate ideas. 

## Law of Demeter

[Law of Demeter](https://en.wikipedia.org/wiki/Law_of_Demeter) is a nearly 30 years old metric, that can be used to determine the coupling of the classes. In general it restricts the list of the objects, that can be called in a method. Lets apply it to the InterestRateCalculator#calculate method from the code below.
{% highlight java %}
public class InterestRateCalculator {
    private RatePolicyFactory ratePolicyFactory;

    @Autowired
    InterestRateCalculator(RatePolicyFactory ratePolicyFactory) {
        this.ratePolicyFactory = ratePolicyFactory;
    }

    public BigDecimal calculate(BigDecimal amount) {
        RatePolicy policy = ratePolicyFactory.getPolicy();
        return policy.calculate(amount);
    }
{% endhighlight %}

In the first line the method is calling a method on object, that is its parameter. There is nothing to worry about. The problem is the second line we are working on the object, that is returned by a call in the first line. And this is against the law of Demeter. Therefore our code can not be considered to be loosely coupled.
      
## The solution

Let's look at the Spring Framework. What is the name of the root interface for accessing a Spring container? The name is BeanFactory. As its name suggest it is responsible for creation of all beans in container. How can we register a factory method that should be used to create the bean. It is very simple. Just add a @Configuration class with a @Bean method.

Let's see some example. Assume, that our product support three different countries. Each of them using different tax policy. Look at the following code:

{% highlight java %}
@Configuration
class TaxPolicyConfiguration {
    @Bean
    TaxPolicy taxPolicy(@Value("${country.code:NONE}") CountryCode countryCode) {
        switch (countryCode) {
            case SE: return new SeTaxPolicy();
            case BE: return new BeTaxPolicy();
            case ME: return new MeTaxPolicy();
        }
        return new DefaultTaxPolicy();
    }
}   
{% endhighlight %}

We can restrict the visibility of the all classes here to package private. The exception is the TaxPolicy interface, that should has public visibility. So REALLY all creation details are hidden from consumer. Due to nature of the Spring components the scan method factory TaxPolicyConfiguration#taxPolicy for creating a tax policy instance would registered in bean factory.  Moreover the container takes resposibility to create the component when it is needed.

## Statefull component - be careful
If you are in need to create a statfull be carefull, there is probably sothing wrong with you archticture. I found another reason to discourage such a solution. There is no way to avoid calling the factory in case you would like to create prototype scope beans, that should be parametrized using a constructor. Look at the code below. There is no way to instate a bean without calling a method on injected.

{% highlight java %}
Component
public class UiGenerator {
    private ApplicationContext appCtx;

    @Autowired
    UiGenerator(ApplicationContext appCtx) {
        this.appCtx = appCtx;
    }

    public void showData() {
        Stream.of(1, 3, 5).forEach(id -> appCtx.getBean(TaxStatementView.class, id).show());
    }
}
{% endhighlight %}

## Sum up
Injection factory brings at least two architectural smells to your. Avoid it, unless you have no human fillings to fairy tales. 
Sample code can be found here [Don't inject factories at github](https://github.com/karolkalinski/dontinjectfactories)
Inspired by [Demeter Saves Mocking Fairies](http://dearjunior.blogspot.com/2009/11/demeter-saves-mocking-fairies.html)
