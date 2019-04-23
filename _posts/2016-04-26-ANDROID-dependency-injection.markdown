---
layout: post
title: 'Una aplicación android utilizando Inyección de dependencias'
date: '2016-04-26 11:45:27'
tags:
- android
categories:
- Android Architecture
---

![di-android](/content/images/2016/4/depen.png){: .center-image }

Después de unas semanas con jornadas de trabajo intensas hoy he vuelto con un tema muy interesante y que desde hace un tiempo he tenido muchas ganas de escribir ya que pienso que es la base y una de las partes fundamentales para escribir un buen software pero en el caso de android su uso no es tan común es por eso que quiero hablar sobre esos beneficios que contrae el usar la inyección de dependencias.

Antes de comenzar con la Inyección de dependencias hay dos principios que debemos conocer para comprender mejor la motivación de usar este patrón y recordar que esto puede ser aplicado en alguna otra plataforma dentro del mundo del software.


## Inversion of Control (IoC)

En los comienzos del desarrollo software cuando los programas eran lineales el flujo de ejecución estaba bajo control absoluto del programador ya que él era la persona que especificaba la secuencia y los procedimientos que se tenían que ejecutar con el paso del tiempo comenzaron a aparecer librerías y frameworks con la finalidad de diseñar de una forma modular y reutilizable lo que dio origen a la inversión de control.

La **IoC (Inversión de control)** es cuando el flujo de ejecución de un software está bajo el control de algún framework o librería y ya no por el desarrollador. En el caso de android la inversión de control se da en los métodos del ciclo de vida de una [Activity](https://developer.android.com/training/basics/activity-lifecycle/starting.html), [Fragment](https://developer.android.com/guide/components/fragments.html) y un [Service](https://developer.android.com/guide/components/services.html) ya que son los que dirigen cómo se deben ir ejecutando ciertas partes de una aplicación. Lo que realmente debemos entender es que todo el software que escribimos nosotros o al menos en android no somos nosotros los desarrolladores quien tenemos el control completo del flujo de ejecución si no el framework y será él mismo quien nos esté diciendo el usuario ha lanzando una activity, el usuario pulsó un checkbox, el usuario pulso un botón, etc.


En un [post del gran Martin Fowler](http://martinfowler.com/bliki/InversionOfControl.html) llama a este fenómeno como el
**Hollywood Principle “Don't call us, we'll call you”** una famosa frase que se les dice a los actores de hollywood tras presentar una audición y que ejemplifica bastante bien a la [IoC](https://es.wikipedia.org/wiki/Inversi%C3%B3n_de_control).

## Dependency Inversion

El principio de Inversión de Dependencias es uno de los cinco componentes de [SOLID](http://devexperto.com/principio-responsabilidad-unica) que si no habías oído hablar de ellos o ya los conoces sabrás que son un conjunto de principios que aplicados correctamente, te ayudarán a escribir software de calidad crearás código que será más fácil de testear y mantener.

La palabra **“inversión”** dentro del principio de **Inversión de Dependencias** viene dentro del término porque se invierte el camino en el cual pensamos típicamente el diseño de la OO ya que los componentes de bajo nivel dependen de las abstracciones de alto nivel.

*"Las clases de alto nivel no deberían depender de las clases de bajo nivel. Ambas deberían depender de las abstracciones."*

La idea de este principio es la reducción de dependencias a clases concretas dentro de nuestra aplicación, es decir el código que es la parte más importante no debe depender de los detalles de implementación como pueden ser algún framework o alguna librería externa que se utilice. Todos estos aspectos se especificarán mediante interfaces y esa parte importante no tendrá que conocer cuál es la implementación real para funcionar.

*"Las abstracciones no deberían depender de los detalles. Los detalles deberían depender de las abstracciones."*

## Coupled classes

Cuando tenemos dos componentes (clases) y uno de ellos depende del otro lo que solemos hacer de forma habitual es crear una instancia dentro de esa clase y hacer lo que tengamos que hacer de una forma muy natural se ve como algo normal y es que en verdad está cumpliendo el objetivo (aunque no de la mejor forma) proveer de sus implementaciones a el componente que las necesite por medio de la creación de una instancia.

**¿Pero cuál es el verdadero problema de hacer esto?**

* El principal problema de hacer esto es que comenzamos a acoplar nuestras clases y el acoplamiento es uno de los mayores problemas del   software, ya que la idea es que una clase no tenga que crear sus propios objetos.
* Nuestras clases no son flexibles porque no podemos sustituir componentes de forma sencilla.
* No se puede testar una clase acoplada si no hay forma de sustituir su comportamiento por test doubles y por ende no se podrá testar una clase acoplada de forma aislada.
* Si es un proyecto en el cual se necesitan cambios constantes la mantenibilidad de nuestro software va  a ser un tanto doloroso.


## What is a Dependency?

Es el acoplamiento entre dos módulos de nuestro código en nuestro caso clases y se manifiesta cuando un módulo necesita de otro para realizar algo.

## Dependency Injection

La inyección de dependencias es un patrón de diseño de software que facilita la inversión de dependencias y se encarga de proveer dependencias en lugar de ser la propia clase quien cree el objeto.
Cuando hablamos de depender de abstracciones estamos haciendo referencia a depender de supertipos que comúnmente se consigue mediante una clase abstracta o una interfaz de las cuales no podemos crear una instancia por lo que tenemos que ser capaces de crear la implementación concreta y pasarsela al objeto que la necesite mediante un constructor o un setter.

La clase **WeatherForecaster** necesita el objeto **LocationService.**

{% highlight java %}

public class WeatherForecaster {

  private final LocationService locationService;

  public WeatherForecaster() {
    this.locationService = new LocationService();
  }
}
{% endhighlight %}

La idea es que una clase no tenga que crear sus propios objetos como lo he mencionado arriba esto puede ser un gran problema al momento que nuestro software comienza a crecer ya que testar y mantener se va volver muy complejo.

{% highlight java %}

public class WeatherForecaster {

  private final LocationService locationService;

  public WeatherForecaster(LocationService locationService) {
    this.locationService = locationService;
  }
}

{% endhighlight %}

Al obtener una dependencia por medio del constructor nos va proveer más flexibilidad al momento de necesitar diferentes implementaciones, durante el testing es una gran ventaja ya que permitirá reemplazar de manera sencilla por algún test double.

En concreto la inyección de dependencias nos permite proveer una dependencia fuera del objeto que la necesita.

## What is a dependency injector?

La Inyección de dependencias en algunas ocasiones puede contraer nuevos problemas y es que si he logrado transmitirte bien los conceptos esperaria que te estés haciendo estas preguntas:

* Si una clase no puede crear sus propios objetos, ¿Entonces debería existir un lugar en donde se crean instancias de los módulos?
* ¿Qué hago si un módulo tiene un montón de dependencias?
* ¿Puedo tener constructores con muchas dependencias?
* ¿Cómo reutilizar dependencias entre objetos ?

Un **inyector de dependencias** es aquel que se va encargar de administrar y proveer todas las dependencias que le configuremos y que haremos uso dentro de nuestra aplicación.

En android especialmente son de gran utilidad y aportan grandes beneficios, ya que nosotros no podemos acceder al constructor de una Activity o un  Fragment por ejemplo lo que inyectar una dependencia mediante un constructor sería casi imposible aun que se puede tambien simular un inyector otras desventajas de no usar un inyector es que nuestro código se vuelve sucio e ilegible más cuando se tiene un exceso de dependencias pasadas por el constructor (si cuando ves clases con muchos “new”) y otra cosa importante es que permite proveer objetos que se utilizan por toda la aplicación de una forma simple.

Yo te recomiendo que utilices el inyector que más te guste y mejor entiendas de preferencia que cumpla con el estándar  *javax.inject annotations (JSR-330)* algunos de los más conocidos que funcionan en android son [Guice](https://github.com/google/guice),  [Dagger](http://square.github.io/dagger/), [Dagger2](http://google.github.io/dagger/)(yo uso este).

**Algo que quiero que tengas en mente y muy claro es que no es obligatorio el uso de un inyector de dependencias para aplicar la inyección de dependencias, ya que solo es una herramienta más que facilita la aplicación de este patrón te recomiendo usar un inyector si en verdad lo necesitas y si lo entiendes completamente (es complicado pero no imposible la curva de aprendizaje de dagger 2 por ejemplo no es del todo simple pero es cuestión de dedicarle tiempo) de lo contrario se podría convertir hasta en un anti patrón.**

## Conclusion

Particularmente para mi estos temas desde mi punto de vista es de la parte más complicada en el mundo del software ya que requiere un nivel de abstracción complejo por eso te invito a que tengas paciencia cuando vayas aprendiendo ;pero una vez que lo entiendes te darás cuenta que la parte más importante de aplicar IoC, Dependency Inversion y Dependency Injection es que te va proveer desacoplamiento,modularidad, mantenibilidad, testabilidad lo que es magnífico para nuestro software haciendo tu día a día como desarrollador más sencillo.

En este post solo quería demostrar y dejar muy claros algunos conceptos antes de utilizar un inyector de dependencias sin tener idea del por qué lo usamos y terminar haciéndolo sólo porque está de moda para mí la parte más interesante es conocer la motivación del por qué usarlo.

Si quieres aprender a usar el inyector de dependencias Dagger2 entonces no te pierdas [mi siguiente artículo](https://erikcaffrey.github.io/ANDROID-kata-dagger2/) que he preparado para tí en donde mediante una kata explico el uso de Dagger2.

### Demo

#### El código está disponible

[Dependency-Injection-Android en Github](https://github.com/erikcaffrey/Dependency-Injection-Android)

![](/content/images/2016/4/battery.png)

## Acknowledgment

Un especial agradecimiento a [Chiu-Ki Chan](https://developers.google.com/experts/people/chiu-ki-chan) Google Developer Expert por su gran trabajo en [Daggerless Dependency Injection for Testing](https://github.com/chiuki/daggerless-di-testing/tree/master) de donde he tomado su genial ejemplo haciendole unas pequeñas modificaciones.

### Resources
* **Martin Fowler** - [InversionOfControl](http://martinfowler.com/bliki/InversionOfControl.html)
* **Chiu-Ki Chan** - [Daggerless Dependency Injection for Testing](https://github.com/chiuki/daggerless-di-testing/tree/master)
* **Jake Wharton** - [The Future of Dependency Injection with Dagger 2](https://www.youtube.com/watch?v=plK0zyRLIP8)
* **Antonio Leiva** - [Dependency Injection-android-dagger](http://antonioleiva.com/dependency-injection-android-dagger-part-1/)
* **Pedro Gómez** - [Dependency Injection on Android](https://www.youtube.com/watch?v=XY2fHxqEBeo)

**"Depend upon abstractions. Do not depend upon concrete classes"**
