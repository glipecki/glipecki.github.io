---
layout: post
published: false
title: How to build and deploy Angular CLI with Spring Boot
---
Every modern web app needs some backend, that's rather bold statement in recent years, but for the sake of simplicity let's assume that's true. If we are talking about backend service for web you probably thought already about REST and if we talk about REST the Spring Boot is obvious framework of choice for many. In this article we will create fresh project based on Spring Boot and Angular CLI. We'll prepare build process for complete deployable artifact and see how to use it in everyday work.

_I assume that you already have basic knowledge of Spring Boot, Angular CLI and Maven. If not, please look at useful links at the bottom._

## Minimal working project

The best way to create new Spring Boot project is to use official Spring Initializer. We can just go to http://start.spring.io/, select all needed components and hit download. But it's not what every geek will do, we will create it like a pro, using curl.

```bash
$ cd demo
$ curl https://start.spring.io/starter.tgz \
  -d groupId=net.lipecki.demo \
  -d packageName=net.lipecki.demo \
  -d artifactId=demo \
  -d dependencies=web \
  | tar -xzvf -

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 50190  100 50104  100    86  54925     94 --:--:-- --:--:-- --:--:-- 54878
x mvnw
x .mvn/
x .mvn/wrapper/
x src/
x src/main/
x src/main/java/
x src/main/java/net/
x src/main/java/net/lipecki/
x src/main/java/net/lipecki/demo/
x src/main/resources/
x src/main/resources/static/
x src/main/resources/templates/
x src/test/
x src/test/java/
x src/test/java/net/
x src/test/java/net/lipecki/
x src/test/java/net/lipecki/demo/
x .gitignore
x .mvn/wrapper/maven-wrapper.jar
x .mvn/wrapper/maven-wrapper.properties
x mvnw.cmd
x pom.xml
x src/main/java/net/lipecki/demo/DemoApplication.java
x src/main/resources/application.properties
x src/test/java/net/lipecki/demo/DemoApplicationTests.java
```

In a result we have fully working Spring Boot app, which we can already build and run without further work. For tests we should add at least one simple REST controller. Just as before, we'll use curl.

```bash
$ curl -L https://goo.gl/MbWM8s -o src/main/java/net/lipecki/demo/GreetingRestController.java
$ cat src/main/java/net/lipecki/demo/GreetingRestController.java 
package net.lipecki.demo;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class GreetingRestController {

	@RequestMapping("/greeting")
	public String greeting() {
		return "Welcome!";
	}

}
```

It's all set and ready to be built using Maven. Based on out preferences we can use our system instance of Maven or locally provided with project Maven Wrapper. The second one allows us to work independently from other projects and simplifies project build pipeline on CI/CD systems. If we decide to use Maven Wrapper it'll download Maven instance upon first run.

```bash
./mvnw clean package
 .
 .
 .
[INFO] --- spring-boot-maven-plugin:1.4.2.RELEASE:repackage (default) @ demo ---
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 4.929 s
[INFO] Finished at: 2016-12-07T20:43:32+01:00
[INFO] Final Memory: 28M/325M
[INFO] ------------------------------------------------------------------------
```

Created artifact is simple jar artifact, which we can run using _java -jar_.

```bash
$ java -jar target/demo-0.0.1-SNAPSHOT.jar 
 .
 .
 .
2016-12-07 20:44:13.392  INFO 70743 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 8080 (http)
2016-12-07 20:44:13.397  INFO 70743 --- [           main] net.lipecki.demo.DemoApplication         : Started DemoApplication in 2.271 seconds (JVM running for 2.643)
```

When app is running we can test our sample REST controller calling its endpoint using curl.

```bash
$ curl http://localhost:8080/greeting && echo
Welcome!
```

## Desirable project structure

_draft_
