---
layout: post
title: 'Model View Presenter en Android'
date: '2015-11-03 20:47:27'
tags:
- android
categories:
- Android Architecture
---

![mvp-android](/content/images/2015/11/mvp_.png){: .center-image }

Estructurar el código de nuestras aplicaciones de una manera que sea mantenible y testable no es del todo simple debido a la forma en que se ha implementando el famoso **Model View Controller**, sin embargo desde hace un tiempo se viene hablando de nuevos caminos para conseguirlo, quizás ya haz escuchado acerca de **Model View Presenter y Model View View Model** por tal motivo quiero hablar sobre estas alternativas para trabajar en la capa de pintado comenzando por MVP y continuando con MVVM en mi [siguiente artículo](https://erikcaffrey.github.io/ANDROID-databinding-android/).

Antes de comenzar a hablar sobre el MVP *recordaremos* la manera en que muchos desarrolladores solemos o solíamos trabajar en el desarrollo de aplicaciones android.

## Model View Controller (MVC)

Recordar que el [MVC](https://teamtreehouse.com/library/build-a-blog-reader-android-app/exploring-the-masterdetail-template/the-modelviewcontroller-mvc-design-pattern-2) es un patrón que permite desacoplar la implementación de las vistas de la implementación de la lógica de negocio de nuestro modelo.

### ¿Cómo se aplica actualmente el MVC en Android?

#### Modelos
El encargado de encapsular y mantener la lógica de negocio lo que tiene que hacer nuestra aplicación (Que vamos a mostrar).

#### Vistas
Los encargados de dibujar las vistas de nuestra aplicación ese conjunto de layouts creados en xml. (Cómo lo vamos a mostrar).

#### Controladores
Su función es intercambiar mensajes entre la vista y el modelo, típicamente se usa una subclase de Activity, Fragment y un Service como un controlador. (Formatea el modelo para su visualización y manipula eventos como la entrada del usuario).

### ¿En donde comienzan los problemas al usar MVC?

Una mala aplicación de este patrón comienza cuando decidimos usar una activity o un Fragment como un controlador ya que las actividades y fragmentos forman parte de la vista dentro de MVC, es decir si son controladores pero únicamente de las vistas, son los encargados de mantener el estado puro y lógico de la vista así como reaccionar ante posibles eventos del usuario.

Si usamos una *Activity como controlador* y su lógica de presentación cambia necesitaremos agregar más código, si a eso le sumamos la lógica de controlador y si también tenemos actividades o fragmentos que hagan lo mismo tendremos que duplicar código porque el controlador no está desacoplado de la vista y es en ese momento en donde comenzamos a realizar una bola de nieve creciente y gigante con nuestro código terminando con clases que contienen miles de líneas de código volviéndose ilegibles e in mantenibles y por supuesto difícilmente podríamos testearlas.


![mvc_tr](/content/images/2015/11/mvc-tr.png){: .center-image }

Esto no quiere decir que el MVC sea malo sino que una mala aplicación de este patrón hace que se convierta en un anti-patrón ya que nos causa muchos problemas o errores en el desarrollo de nuestro día día, normalmente al implementar un controlador dentro de una Activity o un Fragment.

Lo más importante es que siempre debemos intentar separar las vistas de los controladores y es en este punto donde llega MVP ya que permite separar e independizar la mayoría de nuestro código de una forma más elegante.

# Model View Presenter (MVP)

El [Model View Presenter](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93presenter) (MVP) es un patrón derivado del MVC comúnmente usado en la construcción de interfaces gráficas, debemos considerar y tener en mente es que el **MVP** no es un patrón de arquitectura de aplicaciones pero es usado para la arquitectura, es decir sólo se encarga de implementar toda esa lógica de la **capa de presentación**.

**MVP** permite separar la capa de pintado se enfoca en todo lo relacionado con cómo **funciona** la interfaz queda separado del cómo representarlo en nuestra UI, si se aprovecha la flexibilidad que nos ofrece se podría lograr que una misma lógica pudiera tener vistas totalmente diferentes así como mejorar la testabilidad del código de forma independiente.

![mvp](/content/images/2015/11/mvp.png){: .center-image }

#### View

Una View dentro de MVP no representa una vista del SDK de android (Android class View), es decir no es un TextView, ni un Button, ni un Layout, ni una Custom View, ni mucho menos una Activity o un Fragment.

Una View representa una abstracción de que puedo hacer con la vista, normalmente se asocia a una *interface* para representar la funcionalidad de una vista.
La parte importante está en que una Activity o un Fragmento tienen la responsabilidad únicamente de implementar la interface View y conectar las acciones del usuario con el presenter.

#### Presenter

En este patrón es el encargado de coordinar la implementación de la vista y el modelo, actualiza la vista y actúa sobre los eventos de usuario que se envían por la vista. El presenter también recupera los datos del modelo y los prepara para su visualización.

#### Model

Es el proveedor de los datos que queremos mostrar en la vista, si estuviéramos usando **Clean Architecture** el modelo sería un interactor que implemente algún caso de uso.


### ¿Por que deberia usar Model View Presenter?

Para que una aplicación se extendible, mantenible y testable requiere que mantener sus capas separadas, ya que en android existen clases muy acopladas como es el caso de los Cursores y Adaptadores.
Las actividades suelen estar muy acopladas a nuestra interfaz y nuestra lógica de negocio en un MVC clásico, el común ejemplo está cuando decidimos traer información local de algun lugar dentro del dispositivo para pintarla en una simple lista hasta ahí vamos bien ;pero que tal si en unas semanas decidimos traer los datos de la red y olvidar esos datos locales ¿Necesitaremos reescribir esa clase? pues la respuesta es que sí porque nuestra Activity o Fragment estará acoplada a nuestras vistas y a la forma en que obtenemos los datos para pintarlos.

Si aplicamos el **MVP** de una forma limpia la vista jamas sabra de donde se obtienen los datos y solo se enfocara en renderizarlos independizando la vista de la forma de conseguir esos datos.

## Android-Spotify-MVP

Es un ejemplo de Model View Presenter que lo cree con la intención de explicar cómo funciona este patrón dentro de nuestras aplicaciones android también puedes encontrár el código en la parte final del post.

![mvp](/content/images/2015/11/spotify-mvp.png){: .center-image }

### Conclusión

**Implementar el MVP** suele ser muy confuso ya que no existe una camino o una forma estandarizada de cómo se debe implementar en nuestros proyectos, más bien cada uno es responsable de cómo usar el patrón y hacer cumplir la necesidad o el problema que se tenga que resolver.
La clave del patrón varía en función de la cantidad de responsabilidades que deleguemos en el presentador, hacer que nuestras vistas sean lo más tontas posibles esto quiere decir que tomen la menor cantidad de decisiones para que el presenter pueda ser reutilizado además si las entidades del SDK de android quedan fuera de el, será más sencillo hacer unit test.

### Talk UI Patterns

He dado una pequeña charla hablando sobre estos temas que puedes ver aquí [GDG Open Lima Hangout #6 - El Android del Pasado, Presente / Futuro](https://www.youtube.com/watch?v=_e7aACEAfv4).

### Demo

#### El código está disponible

#### Android

[SpotifyMVP en Github](https://github.com/erikcaffrey/SpotifyMVP)

![](/content/images/2015/11/Telecine_2015-11-25-17-19-04.gif)

#### IOS

[MarioKart Kata MVP](https://github.com/erikcaffrey/Swift-ModelViewPresenter)

### Bonus

Si quieres aprender más sobre **MVP** también puedes darle un vistazo al gran trabajo que ha hecho mi buen amigo Pedro Hernández **Pett**, encuentralo en github como **silmood**.

* **MVP + Dagger 2** - [Charla for Android Talks](https://www.youtube.com/watch?v=_yVE1DRY1v8)
* **Introducción al Modelo Vista Presentador en Android** - [Charla for Platzi](https://www.youtube.com/watch?v=qWh1QlRpKxk)
* **Spotify Streamer** - [source code](https://github.com/silmood/Spotify-Streamer)

### Resources
* **Antonio Leiva** - [MVP for Android](http://antonioleiva.com/mvp-android/)
* **Pedro Gómez** - [Effective Android UI](https://www.youtube.com/watch?v=N6yqe88ysNw)
* **Ivan Carballo** - [archi](https://github.com/ivacf/archi)

**"Clean code always looks like it was written by someone who cares. by Michael Feathers"**
