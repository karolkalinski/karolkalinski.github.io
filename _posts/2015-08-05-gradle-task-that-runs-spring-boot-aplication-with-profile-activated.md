---
layout: post
title: Gradle task that runs spring boot aplication with profile activated
---
A good practice, when building a spring application is to have few profiles. Usually everyone adds three common ones: dev, test, prod. Therefore each time you run you application with `gradle` using `runBoot` task  you need to remeber to pass value for system property `spring.profiles.active`  
