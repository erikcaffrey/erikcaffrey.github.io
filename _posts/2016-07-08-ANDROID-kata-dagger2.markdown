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

## Where is Princess Peach?

En el mundo 2 mientras Mario Bros se encontraba en su misión de recolectar monedas y conseguir estrellas, la famosa Princesa Peach fue raptada por un degenerado Bowser quien le ha preparado una serie de planes nada buenos para ella.

Después de un largo camino recorrido en diferentes arenas, mundos y ciudades gracias a la colaboración de Luigi han dado con el paradero de la princesa, para recuperarla esta vez no es necesario que Mario tenga pasar a través de castillos como un bandido destruyendo y derribando cuanto hongo se ponga en el camino o asesinando tortugas lanza martillos, ni mucho menos aniquilando peces voladores, esta vez se trata de ganar una carrera contra el malvado de Bowser.

## Mission

Tu misión es desarrollar una aplicación android que muestre una lista de los competidores utilizando el inyector de dependencias Dagger 2 y con ello ayudar a que Mario Bros pueda conocer las habilidades de sus rivales, sin duda alguna tu colaboración será de utilidad para recuperar a la aterrada Princesa Peach.

# Understanding dagger 2 before to start the challenge

## Introduction to Dependency Injection

En mi [capítulo anterior](https://erikcaffrey.github.io/ANDROID-dependency-injection/) estuve hablando sobre que era la [inversión de control](http://martinfowler.com/bliki/InversionOfControl.html), la inversión de dependencias y [la inyección de dependencias](https://www.youtube.com/watch?v=plK0zyRLIP8), que si no has leído te invito a que lo hagas antes de hacer esta [kata](http://butunclebob.com/ArticleS.UncleBob.TheBowlingGameKata), ya que es fundamental que comprendas el porqué y la motivación de usar un **inyector de dependencias**.

Pues bien hoy es el gran día para aprender [Dagger 2](http://google.github.io/dagger/) mediante un divertido ejemplo y aunque se que usar Dagger 2  es complejo en un principio y difícil de entender, intentaré enseñarte de una forma fácil y espero lograrlo.

## Before of Dagger 2

Los inyectores de dependencias en android comenzaron a tomar fuerza cuando Google lanzó [Guice](https://github.com/google/guice) su propio inyector de dependencias, tiempo después Square vendría a proponer [Dagger 1](http://square.github.io/dagger/) y más tarde Google apostaría por crear [Dagger 2](http://google.github.io/dagger/) tomando como referencia a Dagger 1, aun que no quiero indagar en este tema de la historia (ya que no es el objetivo de esta kata) del por que se fueron creando cada vez una mejor alternativa, si espero que revises esta [presentación](https://docs.google.com/presentation/d/1fby5VeGU9CN8zjw4lAb2QPPsKRxx6mSwCe9q7ECNSJQ/pub?start=false&loop=false&delayms=3000&slide=id.g36c44a5df_0295) que describe perfectamente el cómo llegamos a Dagger 2 y lo más importante es que si
deseas utilizar otro inyector de dependencias asegurate que cuente con el estándar de notaciones [“JSR-330”](https://jcp.org/en/jsr/detail?id=330) ya que permitirá a tu codigo ser reutilizable, testable y mantenible.

## Dagger 2

Es un framework que permite aplicar el patrón de inyección de dependencias en java y android viene a solucionar algunos problemas de otros inyectores y sobre todo con la intención de mejorar la experiencia de los desarrolladores la usar un inyector.

### Features

* Configuración basada en Componentes, adiós a los grafos.
* El código generado es legible y fácil de entender.
* No usa reflexión.
* No se de desarrolla sobre el framework.
* El grafo no es compuesto en tiempo de ejecución.
* Incrementa el performance 13% de acuerdo con Gegrory Kick.
* El grafo es validado en tiempo de build.

Dagger 1 construye su grafo de dependencias en tiempo de ejecución haciendo uso de reflexión para conseguirlo lo que lo volvía lento y difícil de depurar, Dagger 2 soluciona este problema construyendo el grafo en tiempo de compilación y sin reflexión lo que lo hace mas rapido pero menos flexible que Dagger 1 ya que no usa [reflexión](https://en.wikipedia.org/wiki/Reflection_(computer_programming))(si esto es un poco coñazo).

### Setup Dagger 2

Para poder hacer uso de Dagger 2 en android studio necesitamos configurar el procesador de anotaciones simplemente usando el plugin de gradle `android-apt` afortunadamente con la versión `android gradle plugin 2.2` ya que viene contenido.

```gradle
dependencies {
    annotationProcessor 'com.google.dagger:dagger-compiler:2.8'
    compile 'com.google.dagger:dagger:2.8'
    provided 'javax.annotation:jsr250-api:1.0'
}
```

El compilador de Dagger 2 genera código que es usado para crear un grafo, el cual se encargara de resolver nuestras dependencias estas clases son agregadas al class path del IDE durante la compilación usa `provided` para referenciar las dependencias que son necesitadas en tiempo de compilación y `annotationProcessor` procesa las notaciones de nuestras clases  sin agregarlas al [class path](https://docs.oracle.com/javase/tutorial/essential/environment/paths.html).

### Annotations of Dagger 2

* **@Module**

Se usa para marcar una clase como proveedora de dependencias ya que dentro contendrá métodos con la notación **@Provides** de tal forma que al momento de crear una instancia de una clase y esta requiera dependencias dagger sabrá en qué lugar ir a buscar dichos objetos.

* **@Provides**

Cada método utilizando para proveer dependencias deben ser marcados con esta anotación ya que serán los objetos que usará dagger para resolver las dependencias entre nuestras clases. Para evitar el exceso de métodos con esta anotación dagger 2 puede buscar una dependencia en el constructor de la clase simplemente marcándolo con la anotación **@Inject**.

* **@Inject**

Es la forma en que pedimos dependencias y le comunicamos a Dagger que la clase o campo marcado con esta anotación será parte de la inyección de dependencias. Así, Dagger construirá instancias de estas clases anotadas y satisfará sus dependencias.

* **@Component**

Se usa para marcar a una interfaz como un componente ya que es necesario para que Dagger pueda generar código o mejor dicho una clase con el prefijo *Dagger* (Si tu declaras un component ExampleApiComponent dagger te va a generar DaggerExampleApiComponent) esta clase generada es la responsable de instanciar una dependencia de nuestro grafo y usarla para inyectarla en los fields marcados con **@Inject**.

* **@Singleton**

Simplemente se usa para decirle a Dagger que debe crear una sola instancia de un objeto, tal cual el patrón de diseño [singleton](http://www.oodesign.com/singleton-pattern.html).

* **@Named**

Hay veces en que debes proveer una misma dependencia con diferentes implementaciones para ello hay que decirle a dagger que instancia es la que necesitamos que sea inyectada.

Básicamente es una forma de nombrar nuestras dependencias y así poder diferenciarlas se deben nombrar en donde se proveen **@Provides** y en donde se inyectarán **@Inject**.

* **@Scope**

Es de las anotaciones más poderosas de dagger 2 ya que permite crear custom scopes que son algo parecido a un **@Singleton** solo que van relacionadas al ciclo de vida de un componente no de la aplicación. Por ejemplo podría crear un **@LifeActivity** annotation y se usaría para que una instancia viva solo mientras una activity sea requerida.

* **@Qualifier**

El uso de esta anotación tiene sentido cuando tenemos dependencias que se crean de una misma interfaz. Imagina que debes proveer un una instancia de MusicApi y esta tiene dos clases hijas Spotify Api y otra por SoundCloud Api usando qualifier te ayuda a identificar cada una **@SpotifyApi** **@SoundCloudApi** y de esta forma decirle a dagger cual es la que debe proveer ya que ambas son del tipo **MusicApi**. Recuerda que se deben nombrar en donde se proveen **@Provides** y en donde se inyectarán **@Inject**.

### Injections Overview

Ahora que tenemos un panorama de cómo funciona dagger o mejor dicho lo que necesita para funcionar me gustaría que te tomaras un momento para analizar esta imagen que muestra de cómo interactúan cada una de sus piezas para lograr la inyección de dependencias.

![dagger2](/content/images/2016/7/dagger_general.png){: .center-image }

### Putting everything together

Ya que hemos comprendido para qué sirven algunos de los elementos veamos un simple ejemplo para reafirmar el conocimiento adquirido ya que te será de utilidad para resolver la kata que he preparado para tí.

**Objetivo:** Crear una aplicación que muestre una imagen y el nombre de la Princesa Peach.


Empezamos creando el objeto **PrincessPeach** con dos atributos un nombre y una foto de perfil ambos los obtenemos usando resources.

{% highlight java %}
public class PrincessPeach {

  private int name;
  private int photo;

  public PrincessPeach() {
    this.name = R.string.princess_name;
    this.photo = R.drawable.ic_peach;
  }

  public int getName() {
    return name;
  }

  public int getPhoto() {
    return photo;
  }
}
{% endhighlight %}

Simularemos que estamos haciendo una llamada a una api y devolveremos una instancia de tipo **PrincessPeach** con un simple método llamado **getPrincesPeach()**.

{% highlight java %}
public class PrincessPeachApi {

  public PrincessPeach getPrincesPeach() {
    return new PrincessPeach();
  }
}
{% endhighlight %}

**@Module, @Provides**

Definimos un módulo que será el encargado de proveer una instancia de **PrincessPeach**.

{% highlight java %}
@Module public class PrincessPeachApiModule {

  @Provides @Singleton PrincessPeachApi providesPeachApi() {
    return new PrincessPeachApi();
  }
}
{% endhighlight %}

**@Component, @Singleton**

Creamos un componente para indicarle a Dagger en qué lugar debemos inyectar la dependencia y en qué módulo debe ir a buscarla.

{% highlight java %}
@Singleton @Component(modules = { PrincessPeachApiModule.class })
public interface PrincessPeachComponent {

  void inject(PrincessPeachActivity princessPeachActivity);
}
{% endhighlight %}

Es necesario crear una clase que herede de [Application](https://developer.android.com/reference/android/app/Application.html) para que dagger pueda inicializar el grafo y asegurarnos que nuestra instancia está creada durante el ciclo de vida de nuestra aplicación.

Asegúrate dar un rebuild project en caso de que no puedas referenciar el componente de dagger en nuestro caso **DaggerPrincessPeachComponent** (Android Studio, select Build > Rebuild Project).

{% highlight java %}
public class PrincessPeachApplication extends Application {

  private PrincessPeachComponent princessPeachComponent;

  @Override public void onCreate() {
    super.onCreate();
    initializeInjector();
  }

  private void initializeInjector() {
    princessPeachComponent = DaggerPrincessPeachComponent.builder()
        .princessPeachApiModule(new PrincessPeachApiModule())
        .build();
  }

  public PrincessPeachComponent getPrincessPeachComponent() {
    return princessPeachComponent;
  }
}
{% endhighlight %}

Por supuesto también es necesario declarar el nombre de nuestro application en el manifest.

{% highlight xml %}
<application
     android:name=".PrincessPeachApplication"
     android:allowBackup="true"
     android:icon="@mipmap/ic_launcher"
     android:label="@string/app_name"
     android:supportsRtl="true"
     android:theme="@style/AppTheme">
{% endhighlight %}

**@Inject**

Lo único por hacer es inyectar nuestra dependencia en nuestra actividad **PrincessPeachActivity** como se lo indicamos al componente **PrincessPeachComponent** que a su vez este hará uso del módulo **PrincessPeachApiModule** para obtener la instancia de la clase solicitada **PrincessPeachApi**.

{% highlight java %}
public class PrincessPeachActivity extends BaseActivity {

  @Inject PrincessPeachApi princessPeachApi;

  @BindView(R.id.label_peach) MarioKartLabel peachLabel;
  @BindView(R.id.picture_peach) ImageView peachImage;

  @Override protected int getLayoutResID() {
    return R.layout.activity_princess_peach;
  }

  @Override protected void onPrepareActivity() {
    super.onPrepareActivity();
    initializeDagger();
    PrincessPeach princessPeach = getPrincesPeachFromApi();
    renderPrincesPeach(princessPeach);
  }

  private void renderPrincesPeach(PrincessPeach princessPeach) {
    renderName(princessPeach.getName());
    renderPicture(princessPeach.getPhoto());
  }

  private void renderName(@StringRes int name) {
    peachLabel.setText(name);
  }

  private void renderPicture(@DrawableRes int picture) {
    peachImage.setImageDrawable(ContextCompat.getDrawable(this, picture));
  }

  private PrincessPeach getPrincesPeachFromApi() {
    return princessPeachApi.getPrincesPeach();
  }

  private void initializeDagger() {
    PrincessPeachApplication application = (PrincessPeachApplication) getApplication();
    application.getPrincessPeachComponent().inject(this);
  }
}
{% endhighlight %}

<img src="/content/images/2016/7/device-peach-sample.png" alt="Drawing" style="width: 330px;"/>

#### El código está disponible

Este ejemplo puedes encontrarlo en el [mismo](https://github.com/erikcaffrey/Kata-Dagger2-MarioKart) repositorio en el que se encuentra la kata en el *branch* **sample/princess-peach-dagger2** [Sample-Princess-Peach en Github](https://github.com/erikcaffrey/Kata-Dagger2-MarioKart/tree/sample/princess-peach-dagger2).

# Great moment to start the challenge

### Task 1

* Leer sobre [clean architecture](https://erikcaffrey.github.io/ANDROID-clean-architecture/) ya que esta kata sigue sus principios (solo si no sabes nada de Clean Architecture).
* Crea un proyecto nuevo de android en android studio
* Configura tu proyecto para poder hacer uso de la librería de dagger 2

### Task 2

* Implementa tu objeto de dominio de nombre [Character](https://github.com/erikcaffrey/Kata-Dagger2-MarioKart/blob/master/app/src/main/java/io/github/erikcaffrey/kata_dagger2_mariokart/domain/model/Character.java) con los siguientes atributos name, photo, cover, description.
* Implementa un objeto de dominio de nombre [Abilities](https://github.com/erikcaffrey/Kata-Dagger2-MarioKart/blob/master/app/src/main/java/io/github/erikcaffrey/kata_dagger2_mariokart/domain/model/Abilities.java) con los siguientes atributos: accelerate, steer, brake, reverse, lookBehind, drift;
* Agrega a tu objeto Character un atributo de tipo Abilities

### Task 3

* Implementa una clase `FAKE DATA SOURCE` responsable de generar una lista de personajes
* Implementa una clase `REPOSITORY` que tendrá una dependencia con `FAKE DATA SOURCE`
* Implementa un modulo de dagger para proveer una dependencia de tu `REPOSITORY`
* Implementa un componente para especificar en donde se van a inyectar tus dependencias.

### Task 4

* Implementa un `GET ALL CHARACTERS USE CASE` que obtenga `TODOS` los personajes con dependencia a `REPOSITORY`
* Configura tu caso de uso para que dagger pueda inyectar una dependencia de `REPOSITORY`

### Task 5

* Implementa un `CHARACTERS PRESENTER` para mostrar una lista de personajes en una vista con dependencia a `GET ALL CHARACTERS USE CASE`
* Configura tu presenter para que dagger pueda inyectar una dependencia de `GET ALL CHARACTERS USE CASE`

### Task 6

* Implementa una vista android activity, fragment u otra con dependencia a `CHARACTERS PRESENTER`
* Configura tu vista android para que dagger pueda inyectar una dependencia de `CHARACTERS PRESENTER`
* Muestra a los personajes de la forma que mas te guste.

**"Complexity kills. It suck the life out of developers it makes products difficult to plan, build and test. by Ray Ozzie"**

### Resources
* **Gregory Kick** - [DAGGER 2 - A New Type of dependency injection](https://www.youtube.com/watch?v=oK_XtfXPkqw)
* **Fernando Cejas** - [Tasting Dagger 2 on Android](http://fernandocejas.com/2015/04/11/tasting-dagger-2-on-android/)
* **Jake Wharton** - [The Future of Dependency Injection with Dagger 2](https://www.youtube.com/watch?v=plK0zyRLIP8)
* **Daniel Lew** - [Dependency Injection Made Simple](https://realm.io/news/daniel-lew-dependency-injection-dagger/)
* **Pedro Gómez** - [Dependency Injection on Android](https://www.youtube.com/watch?v=XY2fHxqEBeo)

### Demo

#### El código está disponible

[Kata-Dagger2-MarioKart en Github](https://github.com/erikcaffrey/Kata-Dagger2-MarioKart)


![](/content/images/2016/1/Telecine_2017-01-29-22-52-19.gif)
