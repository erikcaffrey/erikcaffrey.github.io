---
layout: post
title: 'Dagger 2 Kata para android developers'
date: '2016-07-08 10:32:10'
tags:
- android
categories:
- Android Architecture
---

![dagger2-android](/content/images/2016/7/mario-kart.png){: .center-image }

En el mundo 2 mientras Mario Bros se encontraba en su misión de recolectar monedas y conseguir estrellas, la famosa Princesa Peach fue raptada por un degenerado Bowser quien le ha preparado una serie de planes nada buenos para ella.

Después de un largo camino recorrido en diferentes arenas, mundos y ciudades gracias a la colaboración de Luigi han dado con el paradero de la princesa, para recuperarla esta vez no es necesario que Mario tenga pasar a través de castillos como un bandido destruyendo y derribando cuanto hongo se ponga en el camino o asesinando tortugas lanza martillos, ni mucho menos aniquilando peces voladores, esta vez se trata de ganar una carrera contra el malvado de Bowser.

## Mission

Tu misión es desarrollar una aplicación android que muestre una lista de los competidores utilizando el inyector de dependencias Dagger 2 y con ello ayudar a que Mario Bros pueda conocer las habilidades de sus rivales, sin duda alguna tu colaboración será de utilidad para recuperar a la aterrada Princesa Peach.

## Introduction to Dependency Injection

En mi [capítulo anterior](https://erikcaffrey.github.io/2016/04/26/dependency-injection/) estuve hablando sobre que era la [inversión de control](http://martinfowler.com/bliki/InversionOfControl.html), la inversión de dependencias y [la inyección de dependencias](https://www.youtube.com/watch?v=plK0zyRLIP8), que si no has leído te invito a que lo hagas antes de hacer esta [kata](http://butunclebob.com/ArticleS.UncleBob.TheBowlingGameKata), ya que es fundamental que comprendas el porqué y la motivación de usar un inyector de dependencias.

Pues bien hoy es el gran día para aprender [Dagger 2](http://google.github.io/dagger/) mediante un divertido ejemplo y aunque se que usar Dagger 2  es complejo en un principio y difícil de entender, intentaré enseñarte de una forma fácil y espero lograrlo.

## Before of Dagger 2

Los inyectores de dependencias en android comenzaron a tomar fuerza cuando Google lanzó [Guice](https://github.com/google/guice) su propio inyector de dependencias, tiempo después Square vendría a proponer [Dagger 1](http://square.github.io/dagger/) y más tarde Google apostaría por crear [Dagger 2](http://google.github.io/dagger/) tomando como referencia a Dagger 1, aun que no quiero indagar en este tema de la historia (ya que no es el objetivo de esta kata) del por que se fueron creando cada vez una mejor alternativa, si espero que revises esta [presentación](https://docs.google.com/presentation/d/1fby5VeGU9CN8zjw4lAb2QPPsKRxx6mSwCe9q7ECNSJQ/pub?start=false&loop=false&delayms=3000&slide=id.g36c44a5df_0295) que describe perfectamente el cómo llegamos a Dagger 2 y lo más importante es que si
deseas utilizar otro inyector de dependencias asegurate que cuente con el estándar de notaciones [“JSR-330”](https://jcp.org/en/jsr/detail?id=330) ya que permitirá a tu codigo ser reutilizable, testable y mantenible.

## Dagger 2

Es un framework que permite aplicar el patrón de inyección de dependencias en java y android viene a solucionar algunos problemas de otros inyectores y sobre todo con la intención de mejorar la experiencia de los desarrolladores la usar un inyector.

### Characteristics

* Configuración basada en Componentes, adiós a los grafos.
* El código generado es legible y fácil de entender.
* No usa reflexión.
* No se de desarrolla sobre el framework.
* El grafo no es compuesto en tiempo de ejecución.
* Incrementa el performance 13% de acuerdo con Gegrory Kick.
* El grafo es validado en tiempo de build.

Dagger 1 construye su grafo de dependencias en tiempo de ejecución haciendo uso de reflexión para la conseguirlo lo que lo volvía lento y difícil de depurar, Dagger 2 soluciona este problema construyendo el grafo en tiempo de compilación y sin reflexión lo que lo hace mas rapido pero menos flexible que Dagger 1 ya que no usa [reflexión](https://en.wikipedia.org/wiki/Reflection_(computer_programming)).

### Task 1

I'm cooking this post

wait for it very soon!

**"Complexity kills. It suck the life out of developers it makes products difficult to plan, build and test. by Ray Ozzie"**

### Resources
* **Gregory Kick** - [DAGGER 2 - A New Type of dependency injection](https://www.youtube.com/watch?v=oK_XtfXPkqw)
* **Fernando Cejas** - [Tasting Dagger 2 on Android](http://fernandocejas.com/2015/04/11/tasting-dagger-2-on-android/)
* **Jake Wharton** - [The Future of Dependency Injection with Dagger 2](https://www.youtube.com/watch?v=plK0zyRLIP8)
* **Daniel Lew** - [Dependency Injection Made Simple](https://realm.io/news/daniel-lew-dependency-injection-dagger/)
* **Pedro Gómez** - [Dependency Injection on Android](https://www.youtube.com/watch?v=XY2fHxqEBeo)

### Demo

#### El código está disponible

I'm cooking this post
I'm cooking this post
I'm cooking this post
I'm cooking this post
I'm cooking this post