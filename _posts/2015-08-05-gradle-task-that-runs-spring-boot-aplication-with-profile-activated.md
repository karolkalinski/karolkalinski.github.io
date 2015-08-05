---
layout: post
title: Gradle task that runs spring boot aplication with profile activated
---
A good practice, when building a spring application is to have few profiles. Usually everyone adds three common ones: dev, test, prod. Therefore each time you run you application with `gradle` using `bootRun` task  you need to remeber to pass value for system property `spring.profiles.active`. However configuring the task to be run with predefined profile could be dangerous. Imagine what could happen if you service is run with profile dev on production. My idea is to create a separate task, that would initialize all properties for developing purposes.

## Preconditions
The code below requires you to apply plugin `spring-boot` to your build process.  

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

## Configuring a task to set up main class name and classpath before execution
In our new task configuration we need to add additional behaviour before task is executed. We add this by calling method `doFirst` with following closure:
{% highlight groovy %}
{
		main = project.mainClassName
		classpath = sourceSets.main.runtimeClasspath
}
{% endhighlight %}
The project property `mainClassName` was set up by one of the `bootRun` dependency the task `findMainClass`, that is a part of the `spring-boot-gradle-plugin` as well. If you take a look at lists of tasks executed before `bootRun`, this one will be there. 

## Configuring a task to set up system properties before execution
The last thing, that is missing is seting up the active spring profiles. We achive it by adding `systemProperty "spring.profiles.active", "dev"` to closure passed to doFirst method. The final code generting new task should be like this:
{% highlight groovy %}

task bootRunDev(type: org.springframework.boot.gradle.run.BootRunTask) {
	doFirst() {
		main = project.mainClassName
		classpath = sourceSets.main.runtimeClasspath
		systemProperty "spring.profiles.active", "dev"
	}
}{% endhighlight %}

Now when you command `./gradlew bootRunDev` in your projects directory, your spring-boot application will be run with `dev` profile activated.
