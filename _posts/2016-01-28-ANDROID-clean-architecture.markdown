---
layout: post
title: 'Aplicando Clean Architecture en Android'
date: '2016-01-28 10:19:16'
tags:
- android
categories:
- Android Architecture
---
![data-binding-android](/content/images/2016/1/clean.png){: .center-image }

## Before starting
Hace ya un tiempo que comencé a desarrollar aplicaciones móviles para android (si cuando android era feo '2.2' y no existía material design) y a lo largo de este proceso he aprendido infinidad de cosas y es que diario aprendo algo nuevo mientras trabajo en mi día a día escribiendo aplicaciones ;pero llegó un momento en el que me sentí preocupado por la calidad de mi código y comencé a cuestionarme:

* ¿Cómo es que se escriben aplicaciones de la calidad?
* ¿Quien escribe el código?
* ¿Que tiene ese código de los ingenieros de google que el mio no tenga?
* ¿Porque sus aplicaciones no truenan?
* ¿Por qué son tan bonitas sus aplicaciones y funcionales más allá de que sean personas con mucho talento?

Me intrigaba saber por qué mi software no era tan bueno y la respuesta la encontré cuando comencé a estudiar temas de [Ingeniería de Software](http://www.oodesign.com/), descubrí las buenas prácticas, [los patrones de diseño](http://www.amazon.com/gp/product/0201633612/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=0201633612&linkCode=as2&tag=wwwconsultguc-20&linkId=B53D2MJ6G5FL7J5R), las arquitecturas y los test, etc.
En ese momento realmente me di cuenta que había un mundo detrás que iba más allá de pantallas, controles y menús, me obsesione mucho que comencé a investigar y estudiar más para poder darle la respuesta a cada una de mis preguntas y aunque seguiré en ese proceso de mejora todo el tiempo esta vez quiero debatir ese camino que hace más [saludable al software](http://www.amazon.com/gp/product/0135974445/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=0135974445&linkCode=as2&tag=wwwconsultguc-20&linkId=VJEGNW65CZHRRPTY) y para ello me gustaría que pensaras en esta frase antes de comenzar a escribir código **“Clean code always looks like it was written by someone who cares”** porque escribir buen código es nuestra responsabilidad como ingenieros.

Hay que tener en cuenta que escribir software de calidad es muy complejo pero el punto clave está en que para construir un software de calidad necesitas conocer sobre ingeniería de software como lo he mencionado arriba, conocer metodologias de desarrollo, principios de la programación, conocer patrones de diseño más no reinventar la rueda y sobre todo una base es decir una Arquitectura de software es fundamental porque te van a obligar a separar tu software en capas y eso beneficiara a que escribas código de forma legible, mantenible y testable. Tu no puedes dar **mantenimiento** o hacer cambios a una aplicación móvil si tu código no es mantenible, tu no puedes escribir **test** si tu no tienes una arquitectura que te permita hacerlo.

Con el paso del tiempo y en mi proceso de mejorar la calidad de mi software conocí [“The Clean Architecture”](https://blog.8thlight.com/uncle-bob/2012/08/13/the-clean-architecture.html) gracias a un blog post muy famoso del gran [Tío Bob](https://twitter.com/unclebobmartin) (Robert Cecil Martin) y es que es algo de lo que se ha venido hablando en los últimos años bastante dentro de la comunidad android principalmente en España y parte de Europa.


**Voy a comenzar describiendo lo que he aprendido  a lo largo de este tiempo revisando code examples, charlas y blog posts de Clean Archictecture de diversas fuentes.**  

# The Clean Architecture

Surge con la problemática que en la mayoría de los desarrollos en los cuales existe un framework se encuentra más código que lo representa, en lugar del problema que está resolviendo, es decir la idea es centrarse en lo que tiene que hacer la aplicación como tal y es por ello que dice que debemos de pensar en él framework como el mecanismo de paso de mensajes hacia nuestro software, el cual se debe mirar como una herramienta o plugin de lo que queremos realizar.

El autor dice en varias de sus [charlas](https://www.youtube.com/watch?v=Nltqi7ODZTM) que más que una arquitectura sólo son una serie de condiciones que se deben cumplir para que una arquitectura de considere **”clean”**, solo son una serie de reglas que lo único que van a hacer es dividir el software en capas.

![clean_archi](/content/images/2016/1/clean_archi.png){: .center-image }

Antes de comenzar a describir los componentes importantes del esquema debes tener en cuenta que no es obligatorio ni mucho menos mandatorio usar solo 4 círculos ya que de acuerdo a Uncle Bob solo son esquemáticos y quizás tu problema necesita de más para poder ser resuelto.

Lo único que nos dice es que hay que respetar la **Dependency Rule**: *“El código fuente sólo puede apuntar hacia adentro“*.

### Entities

Son aquellos objetos que van a representar los actores importantes de la lógica de negocio de nuestra aplicación.

### Use Cases

Están muy relacionados con las entidades y son los encargados de implementar lógica de negocio de nuestra aplicación, ya que van a orientar todas aquellas interacciones del flujo de datos y entidades dejando al framework fuera de todo esto (en nuestro caso el sdk android), también se conocen como interactores.

*(En ingeniería de software un caso de uso se define como: “Una secuencia de interacciones que se encargan de modelar alguna acción que quiero que mi software realice”)*

### Interface Adapters

Convierten los datos en el formato más conveniente para los casos de uso y entidades. (De manera recurrente aqui encontraras controladores y presentadores.)

### Frameworks and Drivers

Aquí es donde residen los detalles y todo ese conjunto de plataformas externas y herramientas como puede ser la UI, Web, DB , Devices, etc. Generalmente solo se deben comunicar con el siguiente círculo a su interior.

## Android Clean Architecture

![android_archi](/content/images/2016/1/android_archi.png){: .center-image }

Este esquema es una representación de cómo se aplica el **Clean Architecture** en una aplicación **android** ya que gracias a la gran participación de muchos desarrolladores de la comunidad android alrededor del mundo se ha ido mejorando en estos últimos meses ya que se le han agregado dos componentes importantes el primero un patrón en la capa de presentación que puede ser [MVP](https://erikcaffrey.github.io/2015/11/03/mvp/), [MVVM](https://erikcaffrey.github.io/2015/12/16/databinding-android/), MVC o el que prefieras y el segundo se ha agregado el [Repository Pattern](http://martinfowler.com/eaaCatalog/repository.html) en la capa de datos para abstraer el origen de datos y destacar que estos patrones no están dentro de la arquitectura que **Uncle Bob** describe, es trabajo de una comunidad con el único objetivo de mejorar el desarrollo de nuestras aplicaciones android y si estoy seguro que te encontraras diferentes implementaciones porque realmente no hay un camino a seguir pero particularmente es la que más me gusta y es la que se ha adaptado a mis problemas.

He escrito un ejemplo para demostrar el uso de esta arquitectura el cual he llamado [Euro-CleanArchitecture](https://github.com/erikcaffrey/Euro-CleanArchitecture) y todo el código está disponible en **github**.

### Presentation Layer

Es la capa en donde ocurre todo lo relacionado a cómo funcionan la vistas normalmente activities y fragmentos los cuales contienen lógica pero únicamente enfocada en cómo funciona la vistas es decir el que debo mostrarle al usuario y cuando debe hacerlo.
Para lograr un mejor manejo de vistas en esta capa se suelen usar patrones de UI (en el ejemplo que he escrito encontrarás MVP).

Yo he escrito un post acerca del [MVP](https://erikcaffrey.github.io/2015/11/03/mvp/) y otro sobre [MVVM](https://erikcaffrey.github.io/2015/12/16/databinding-android/) si solo has utilizado MVC dale una mirada a estos posts por que te ayudarán a separar tus vistas de la lógica de negocio de una forma elegante.


### Domain Layer

Toda la lógica de negocio ocurre en esta capa aquí solo lo enfocado a que tiene que hacer nuestra aplicación. Se encuentran entidades y casos de uso regularmente es solo código java (puede ser kotlin o scala es indiferente) y no hay ninguna dependencia android.

### Data Layer

Es la capa en donde se obtienen todos los datos que necesita nuestra aplicación para funcionar y los datos pueden ser de proveídos por una base de datos, de la red o de la memoria o de donde nos imaginemos particularmente en android hacemos uso de [Repository Pattern](http://martinfowler.com/eaaCatalog/repository.html) un patrón que permite abstraer el origen de datos en donde no va importar de donde vengan los datos lo importante es que seran obtenidos de algún lugar y podremos utilizarlos para hacer las acciones que tengamos que hacer.

Algo que valdría la pena mencionar es que cada capa tiene su propio modelo de datos ya que será más fácil de testar y modificar para lo que estaremos usamos mappers para estar cambiando objetos de datos a un objeto de dominio y de dominio a un objeto de nuestra vista no quiero indagar mucho aquí pero cuando veas el código te darás cuenta de lo útil que es.

### Benefits of using Clean Architecture

Existe una cantidad muy grande de diversas arquitecturas algunas en las cuales Uncle Bob se ha basado son [Hexagonal Architecture](http://alistair.cockburn.us/Hexagonal+architecture), [Onion Architecture](http://jeffreypalermo.com/blog/the-onion-architecture-part-1/), [Screaming Architecture](http://blog.8thlight.com/uncle-bob/2011/09/30/Screaming-Architecture.html) y lo importante es que cada una de ellas varía en sus detalles de implementación pero todas son similares y tienen el mismo objetivo separar en capas nuestro software con la finalidad de darle flexibilidad.

Las principales aportaciones que esta arquitectura seguirá motivando son:

* Independiente de frameworks
* Testable
* Independendiente de nuestra UI
* Independendiente de nuestra DB
* Independendiente de cualquier herramienta externa

## Clean-Architecture-Android

Es un ejemplo de Clean Architecture que lo cree con la intención de explicar cómo funciona esta arquitectura dentro de nuestras aplicaciones android también puedes encontrár el código en la parte final del post.

![mvp](/content/images/2016/1/euro_app.png){: .center-image }

### Conclusion

Como ya lo he mencionado arriba solo quedarnos con la idea que Clean Architecture sólo son una serie de reglas a cumplir para crear software por capas dando origen a un código flexible, mantenible y testable. Otro aspecto importante que debemos tener en mente  es que no hay camino a seguir que sea el correcto para implementar esta arquitectura  crea la implementación que mejor te guste y que resuelva tu problema de la mejor forma pero no olvidar generar código que esconda detalles de implementación.


**"Bad code affect to your Customer. by Martin Fowler"**

### Resources
* **Robert Martin (Uncle-bob)** - [The Clean Architecture](https://blog.8thlight.com/uncle-bob/2012/08/13/the-clean-architecture.html)
* **Robert Martin (Uncle-bob)** - [Architecture the Lost Years](https://www.youtube.com/watch?v=WpkDN78P884)
* **Karumi** - [Rosie (is an Android framework to create applications following the principles of Clean Architecture)](https://github.com/Karumi/Rosie)
* **Karumi** - [Clean Architecture Hangouts (karumi is the biggest contributors to Clean Architecture with wonderful projects, talks and post.)](https://www.youtube.com/watch?v=31lWMsCeoSA)
* **Fernando Cejas** - [Android-CleanArchitecture (Github Example and Blog Posts)](https://github.com/android10/Android-CleanArchitecture)
* **Jorge Barroso** [Forgetting Android](https://www.youtube.com/watch?v=ROdIvrLL1ao)

### Demo

#### El código está disponible

[Clean-Architecture-Android en Github](https://github.com/erikcaffrey/Euro-CleanArchitecture)

![](/content/images/2016/1/Telecine_2016-04-11-09-46-01.gif)
