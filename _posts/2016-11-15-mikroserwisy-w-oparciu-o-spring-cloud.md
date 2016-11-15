---
layout: post
published: false
title: Mikroserwisy w oparciu o Spring Cloud
---
Wszyscy wiemy jak kapryśna jest nasza branża, programiści z dnia na dzień zakochują się w nowych technologiach i równie szybko je porzucają. Ciągle dążymy za prostym rozwiązaniem rozwiązujące wszystkie nasze problemy. Jednak w kilku ostatnich latach jeden z trendów wydaje się utrzymywać niesłabnące zainteresowanie - architektura microservices. Wystarczy prześledzić kilka ostatnich konferencji (chociażby devoxx poland), lokalnych JUGów czy vJUGów.

Jest to o tyle ciekawe, że mówimy o rozważaniach w dużej mierze teoretycznych, z pogranicza architektury. Kolejne prezentacje wprowazdają nas w koncepcje, mówią o zaletach i rozważają filozoficzne pytanie - "jak duży powinien być mój mikroserwis". O ile teorii mieliśmy okazję złapać już dużo (chociażby od [Martina Fowlera](http://www.martinfowler.com/articles/microservices.html)), to z praktyką bywa różnie. Oczywiście mamy [Adama Biena](http://www.adam-bien.com/roller/abien/), który nie boi się mierzyć z tematem na scenie, co jednak z fanami Springa?

W dalszej części wpisu przedstawię przygodę z architekturą mikro usług w oparciu o Spring i Spring Cloud. Zobaczymy z jakimi typowymi problemami/zagadnieniami będziemy się mierzyć oraz jakie narzędzia znajdziemy w Springowym toolboxie.

## Z czym będziemy się mierzyć?

Myśląc o microservies wielu z nas potrafiłoby jednym tchemy wyrecytować kilka obowiązkowych punktów:
- małe usługi, najlepiej utrzymywalne przez zespoły mierzone w pizzach,
- dowolność w wyborze technologi,
- luźno powiązane usługi,
- rozdzielność usług na etapie wdrażania i wykonywania,
- prosty protokół komunikacyjny, preferowany HTTP, pewnie REST JSON
- itp.

Często wymieniane punkty skupiają się na zaletach i korzyściach płynących z wdrożenia nowej architektury.

Jednak migracja z monolitu do mikro usług to nie tylko same korzyści, pojawia się też zupełnie niespotykana wcześniej klasa problemów. Od teraz będziemy projektować i wdrażać rozproszony system, to sprawia że mamy teraz nowe wyzwania:
- okresowa niedostępność usług
- wiele instancji usług
- monitorowanie rozproszonych operacji
- brak możliwości zatwierdzenia operacji na poziomie systemu
- rozproszona konfiguracja i konieczność jej synchronizacji
- niezawodna komunikacja pomiędzy usługami, model zdarzeń w systemie
- synchronizacja operacji, które nie mogą być realizowane równocześnie
- znaczne skomplikowanie wdrożenia wielu zmieniających się elementów
- ukrycie złożoności systemu dla aplikacji klienckich

Nic jednak tak nie motywuje programistów jak nowe wyzwania! W kolejnych punktach przyjrzymy się gotowym rozwiązaniom wymienionych zagadnień.

## Pierwsza usługa

- w oparciu o spring boot (jar)
- łatwy start i wystawienie
- łatwe uruchomienie i deployment (w tym wielu instancji obok siebie)

## xxx

## yyy

## zzz

## tl;dr

xxx

- okresowa niedostępność usług -> Hystrix, LoadBalancer
- wiele instancji usług - > ServiceDiscovery, LoadBalancer
- monitorowanie rozproszonych operacji -> Zipkin, Sleuth
- brak możliwości zatwierdzenia operacji na poziomie systemu -> eventually consistent, messaging
- rozproszona konfiguracja i konieczność jej synchronizacji -> config server, consul
- niezawodna komunikacja pomiędzy usługami, model zdarzeń w systemie -> service discovery, loadbalancing, api gateway?, messaging, stream
- synchronizacja operacji, które nie mogą być realizowane równocześnie -> leader election
- znaczne skomplikowanie wdrożenia wielu zmieniających się elementów -> spring boot, aplikacja jako jar
- ukrycie złożoności systemu dla aplikacji klienckich -> api gateway

