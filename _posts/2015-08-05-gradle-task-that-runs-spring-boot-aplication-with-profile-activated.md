---
layout: post
title: Gradle task that runs spring boot aplication with profile activated
---
A good practice, when building a spring application is to have few profiles. Usually everyone adds three common ones: dev, test, prod. Therefore each time you run you application with `gradle` using `bootRun` task  you need to remeber to pass value for system property `spring.profiles.active`. However configuring the task to be run with predefined profile could be dangerous. Imagine what could happen if you service is run with profile dev on production. My idea is to create a separate task, that would initialize all properties for developing purposes.

## Preconditions
The code below requires you to apply plugin `spring-boot` to your build process. If you decide to define new task in module imported from your main gradle script, you need to apply the plugin to this module.   

## Creating a task in gradle that extends another
Creating a task in gradle that extends another one is terrbile simple. The following line would create `newTask` extending `ExtendedClass`:
{% highlight groovy %}
  task newTask(type: ExtendedClass)
{% endhighlight %}
In our case we would like to extend task `org.springframework.boot.gradle.run.BootRunTask` that is part of the `spring-boot-gradle-plugin`. This task runs the project with support for auto-detecting main class and reloading static resources. Lets choose `bootRunDev` name for out new task. Therefore the line to be placed in in your `build.gradle` shoud be:
{% highlight groovy %}
  task bootRunDev(type: org.springframework.boot.gradle.run.BootRunTask)
{% endhighlight %}
The attemp to use this command to start application results in error:
{% highlight vctreestatus %}
FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':execute'.
> No main class specified
{% endhighlight %}
