---
layout: post
published: false
title: Budowanie aplikacji Angular CLI + Spring Boot
---
Każda nietrywialna aplikacja potrzebuje backendu. O ile obecnie to nie jest prawda, to na potrzeby tego artykułu przyjmijmy, że tak jest. A jak backend współpracujący z aplikacją webową to _REST_, a jak _REST_ to _Spring_ i _Spring Boot_. W kilku kolejnych akapitach stworzymy i z sukcesem przygotujemy gotowy do wdrożenia artefakt składający się z aplikacji webowej w _Angular 2_ i backendu usługowego wykorzystującego _Spring Boot_.

## Stworzenie minimalnego projektu

Nowy projekt najłatwiej stworzymy wykorzystując _Spring Initializer_, możemy to zrobić wchodząc [http://start.spring.io/]() i wyklikując konfigurację projektu, lub możemy to zrobić w stylu prawdziwego geeka - _curlem_.

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

W efekcie dostajemy minimalną aplikację _Spring Boot_, którą możemy zbudować i uruchomić. Do testów dorzućmy prosty kontroler.

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

Całość możemy już zbudować

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

uruchomić

```bash
$ java -jar target/demo-0.0.1-SNAPSHOT.jar 
 .
 .
 .
2016-12-07 20:44:13.392  INFO 70743 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 8080 (http)
2016-12-07 20:44:13.397  INFO 70743 --- [           main] net.lipecki.demo.DemoApplication         : Started DemoApplication in 2.271 seconds (JVM running for 2.643)
```

i przetestować

```bash
$ curl http://localhost:8080/greeting && echo
Welcome!
```

## Docelowa struktura projektu

W przypadku najprostszego podejścia wystarczające byłoby dorzucenie tylko kodów części webowej do aktualnego szkieletu i zadbania o jego poprawne budowanie. Jednak dla nas wystarczające to za mało, od razu przygotujemy strukturę która będzie miała szansę wytrzymać próbę czasu.

Minimalny sensowny podział to przygotowanie dwóch modułów:

- artefaktu wdrożeniowego z usługami REST,
- aplikacji webowej.

W tym celu:

- dodajemy do projektu dwa moduły,
 - demo-app,
 - demo-web,
- przenosimy dotychczasową konfigurację budowania i kody do demo-app.

Całość zmian możemy zweryfikować w commicie do testowego repozytorium _GitHub_: [https://goo.gl/XvgmU8]().

Po tych zmianach, nadal możemy budować i uruchamiać aplikację, jednak tym razem z poziomu modułu demo-app.

```bash
$ ./mvnw clean package
 .
 .
 .
[INFO] Reactor Summary:
[INFO] 
[INFO] demo ............................................... SUCCESS [  0.141 s]
[INFO] demo-app ........................................... SUCCESS [  4.909 s]
[INFO] demo-web ........................................... SUCCESS [  0.002 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 5.329 s
[INFO] Finished at: 2016-12-07T21:15:51+01:00
[INFO] Final Memory: 30M/309M
[INFO] ------------------------------------------------------------------------

$ java -jar demo-app/target/demo-app-0.0.1-SNAPSHOT.jar
 .
 .
 .
2016-12-07 21:16:30.569  INFO 71306 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 8080 (http)
2016-12-07 21:16:30.574  INFO 71306 --- [           main] net.lipecki.demo.DemoApplication         : Started DemoApplication in 2.732 seconds (JVM running for 3.117)
```

## Dodanie i budownie części web

Projekt części webowej najłatwiej stworzyć z wykorzystaniem Angular CLI, w tym celu wykonujemy:

```bash
$ pwd
/tmp/demo/demo-web/
$ rm -rf src
$ ng init --style=scss
 .
 .
 .
Installing packages for tooling via npm.
Installed packages for tooling via npm.
```

Dla tak przygotowanego projektu musimy jeszcze zmienić standardową konfigurację folderu budowania z _dist_ na _target/webapp_: [https://goo.gl/w5plW2]().

W tym momencie możemy już swobodnie pracować z aplikacją uruchamiając ją za pomocą _ng serve_. Kolejnym krokiem będzie zintegrowanie procesu budowania _ng build_ z budowaniem modułu _Maven_. W tym celu wykorzystamy plugin _frontend-maven-plugin_.

Plugin _frontend-maven-plugin_ pozwala:

- zainstalować niezależną od systemowej wersję _node_ i _npm_,
- uruchomić instalację zależności _npm_,
- uruchomić budowanie aplikacji za pomocą _ng_.

Całość konfiguracji wprowadzamy definiując dodatkowe elementy _execution_ konfiguracji pluginu.

Instalowanie node i npm we wskazanej wersji:

```xml
<execution>
  <id>install node and npm</id>
  <goals>
    <goal>install-node-and-npm</goal>
  </goals>
  <configuration>
    <nodeVersion>v6.9.1</nodeVersion>
  </configuration>
</execution>
```

Instalowanie zależności _npn_:

```xml
<execution>
  <id>npm install</id>
  <goals>
    <goal>npm</goal>
  </goals>
</execution>
```

Uruchomienie budowania aplikacji:

```xml
<execution>
  <id>node build app</id>
  <phase>prepare-package</phase>
  <goals>
    <goal>npm</goal>
  </goals>
  <configuration>
    <arguments>run-script build</arguments>
  </configuration>
</execution>
```

Na sam koniec musimy jeszcze zdefiniować używany wcześniej skrypt npm - build. W pliku package.json dodajemy:

```json
"scripts": {
  "build": "node node_modules/angular-cli/bin/ng build"
}
```

Tak przygotowana konfiguracja pozwala zintegrować budowanie aplikacji web z fazami cyklu życia _Maven_. Dodatkowo dostajemy uspójniony sposób uruchomienia za pomocą polecenia _npm build_. Dzięki wykorzystaniu _frontend-maven-plugin_ uniezależniamy proces budowania od środowiska, wszystkie wymagane biblioteki (_node_, _npm_, _angular-cli_) są instalowane i wykonywane lokalnie w folderze projektu.

Całość zmian z tego kroku możemy obejrzeć w commicie _GitHub_: [https://goo.gl/HgokqV]().

## Składanie artefaktu z częścią web

Kolejnym krokiem jest umożliwienie spakowania modułu odpowiedzialnego za część web do pojedycznego artefaktu, gotowego do wykorzystania jako zależność lub przesłania do repozytorium artefaktów. W tym celu wykorzystamy plugin _maven-assembly-plugin_, dla którego określamu wynikową nazwę artefaktu oraz plik opisujący sposób budowania artefaktu.

Konfiguracja _maven-assembly-plugin_ w _pom.xml_ modułu web:

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-assembly-plugin</artifactId>
      <configuration>
        <descriptor>assembly.xml</descriptor>
        <finalName>demo-web-${project.version}</finalName>
        <appendAssemblyId>false</appendAssemblyId>
      </configuration>
    <executions>
      <execution>
        <phase>package</phase>
          <goals>
            <goal>single</goal>
          </goals>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```

Definicja sposobu budowania artefaktu w _assembly.xml_:

```xml
<assembly>
  <id>zip</id>
  <formats>
    <format>zip</format>
  </formats>
  <includeBaseDirectory>false</includeBaseDirectory>
  <fileSets>
    <fileSet>
      <directory>target/webapp</directory>
      <outputDirectory></outputDirectory>
      <includes>
        <include>**</include>
      </includes>
    </fileSet>
  </fileSets>
</assembly>
```

Całość możemy przetestować wykonując:

```bash
$ pwd
/tmp/demo
$ ./mvnw clean package
 .
 .
 .
$ ls demo-web/target/demo-web-0.0.1-SNAPSHOT.zip 
demo-web/target/demo-web-0.0.1-SNAPSHOT.zip
```

Commit zawierający zmiany: [https://goo.gl/jL0tca]().

## Składanie artefaktu wdrożeniowego

Ostatnie co musimy zrobić żeby nasza aplikacja składała się w pojedynczy wykonywalny artefakt to skonfigurować moduł _demo-web_ jako zależność w projekcie _demo-app_ oraz skonfigurowanie pluginu _maven-dependency-plugin_, który będzie odpowiadał za odpowiednie rozpakowanie zasobów.

Definiujemy zależność:

```xml
<dependencies>
  <dependency>
    <groupId>net.lipecki.vote</groupId>
    <artifactId>vote-web</artifactId>
    <version>${project.version}</version>
    <type>zip</type>
  </dependency>
</dependencies>
```

i konfigurujemy plugin _maven-dependency-plugin_ tak, żeby zawartość modułu _web_ trafiała do wynikowego artefaktu:

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-dependency-plugin</artifactId>
      <executions>
        <execution>
          <id>unpack</id>
          <phase>generate-resources</phase>
          <goals>
            <goal>unpack</goal>
          </goals>
          <configuration>
            <artifactItems>
              <artifactItem>
                <groupId>net.lipecki.demo</groupId>
                <artifactId>demo-web</artifactId>
                <version>${project.version}</version>
                <type>zip</type>
              </artifactItem>
            </artifactItems>
            <outputDirectory>${project.build.directory}/classes/resources</outputDirectory>
          </configuration>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```

W tym momencie mamy już gotową kompletną konfigurację budowania aplikacji. Po zbudowaniu i uruchomieniu aplikacji możemy zarówno wywołać testową usługę REST, jak i obejrzeć szkielet aplikacji Angular 2.

```bash
$ ./mvnw clean package
 .
 .
 .
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary:
[INFO] 
[INFO] demo ............................................... SUCCESS [  0.120 s]
[INFO] demo-web ........................................... SUCCESS [ 18.459 s]
[INFO] demo-app ........................................... SUCCESS [  5.797 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 24.644 s
[INFO] Finished at: 2016-12-07T22:06:14+01:00
[INFO] Final Memory: 37M/346M
[INFO] ------------------------------------------------------------------------

$ java -jar demo-app/target/demo-app-0.0.1-SNAPSHOT.jar
 .
 .
 .
2016-12-07 22:08:02.937  INFO 72316 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 8080 (http)
2016-12-07 22:08:02.942  INFO 72316 --- [           main] net.lipecki.demo.DemoApplication         : Started DemoApplication in 2.681 seconds (JVM running for 3.077)

$ curl http://localhost:8080/
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
  <title>DemoWeb</title>
  <base href="/">

  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="icon" type="image/x-icon" href="favicon.ico">
</head>
<body>
  <app-root>Loading...</app-root>
<script type="text/javascript" src="inline.bundle.js"></script><script type="text/javascript" src="styles.bundle.js"></script><script type="text/javascript" src="vendor.bundle.js"></script><script type="text/javascript" src="main.bundle.js"></script></body>
</html>
```

Komplet dotychczasowych zmian możemy podsumować w repozytorium _GitHub_: https://goo.gl/AvUldp.

## Obsługa routingu z wykorzystaniem history.pushState (html5 url style)

Poza samym budowaniem i uruchamianiem aplikacji, warto jeszcze zadbać o wsparcie dla nowych sposobów nawigacji. Dotychczas aplikacje web można było łatwo rozpoznać po routingu opartym o #... w url. Taki sposób nawigacji nie narzuca żadnych ograniczeń na stronę serwerową aplikacji, jednak tworzy kilka niemożliwych do rozwiązania problemów, np. renderowanie po stronie serwerowej, czy wsparcie dla SEO.

Obecnie, większość nowoczesnych przeglądarek dostarcza nowe _API_ _history.pushState_ pozwalające zrealizować nawigację z pominięciem znaku #. Po szczegóły odsyłam do oficjalnej dokumentacji _Angular_, natomiast w kolejnych akapitach zajmiemy się konfiguracją Spring Boot wspierającą tę strategię.

W założeniu przedstawiony problem możemy uprościć do zwracania treści _index.html_ zawsze wtedy, kiedy standardowo zwrócilibyśmy błąd _404_. Rozwiązanie powinno uwzględniać zarówno istnienie zdefiniowanych w aplikacji mapowań, jak i pobieranie zasobów dostępnych w lokalizacjach zasobów statycznych.

Najprostszym rozwiązaniem jest zdefiniowanie własnego _ErrorViewResolver_, który dla błędów _404_ wykona przekierowanie na zasób _/index.html_.

```java
@Bean
public ErrorViewResolver customErrorViewResolver() {
  final ModelAndView redirectToIndexHtml = new ModelAndView("forward:/index.html", Collections.emptyMap(), HttpStatus.OK);
    return (request, status, model) -> status == HttpStatus.NOT_FOUND ? redirectToIndexHtml : null;
}
```

Wykorzystanie własnego _ErrorViewResolver_ dodatkowo zapewnia nam wsparcie dla rozróżniania żądań na podstawie nagłówka _HTTP_ _produces_. To znaczy, że żądania z przeglądarek (zawierające produces = "text/html") zostaną obsłużone zawartością zasobu _/index.html_, natomiast pozostałe (np. curl) odpowiedzą standardowym błędem _404_.

## Możliwe rozszerzenia

Wartym rozważenia rozszerzeniem projektu może być wydzielenie trzeciego modułu i podzielenie dodatkowe aplikacji:

```
demo
\-- demo-rest
\-- demo-app
\-- demo-web
```

Gdzie moduły:

- demo-web - zawiera zasoby aplikacji web,
- demo-rest - zawiera samodzielnie uruchamialną aplikację dostarczającą komplet usług _REST_,
- demo-app - jest złączeniem modułów web i rest w jeden wykonywalny artefakt.

Ostateczną wersję aplikacji możemy obejrzeć na GitHub: [https://github.com/glipecki/spring-with-angular-cli-demo](spring-with-angular-cli-demo@github).

## Podsumowanie

Jeżeli na codzień pracujesz z projektami opartymi o _Spring Boot_ i _Maven_ ich integracja z aplikacjami pisanymi w _Angular CLI_ nie będzie stanowić dużego wyzwania. W podstawowej realizacji pomoże Ci plugin _maven-frontend-plugin_, natomiast wykorzystując dodatkowo _maven-assembly-plugin_ i _maven-dependency-plugin_ możliwe jest przygotowanie dużo bardziej złożonych procesów budowania aplikacji.

## Materiały
- https://github.com/glipecki/spring-with-angular-cli-demo
- http://www.consdata.pl/blog/7-szybki-start-z-angular-cli
- https://angular.io/docs/ts/latest/guide/router.html#!#browser-url-styles
