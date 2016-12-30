---
layout: post
title: '¿Por qué deberíamos usar un RecyclerView?'
date: '2015-10-05 18:13:27'
tags:
- android
categories:
- Android Ui
---

![recycler-android](/content/images/2015/10/recycler_view_concep.jpg){: .center-image }

El día de hoy quiero incitar al debate y mostrar mediante algunos ejemplos la importancia que tiene usar un [RecyclerView](https://developer.android.com/training/material/lists-cards.html) he visto personas que aún se rehúsan a usarlo y otras más que no aprovechan al máximo todos los recursos que provee.

Sabemos que muchas personas están familiarizadas con este componente debido a que fue integrado al SDK de android con la llegada de la nueva versión android 5.0 "L" "Lollipop", la cual fue anunciada en el [Google I/O 2014](https://www.youtube.com/watch?v=wtLJPvx7-ys) lo que significa que estamos hablando que esto tiene ya poco más de un año.

Pues bien ya has tenido tiempo para explorar a fondo este nuevo control, si lo has usado me gustaria que te preguntaras:

* **¿Ya utilizas RecyclerView en tus proyectos, has dado el cambio y dejaste de usar ListView o GridView?**.
* Si si lo usas **¿Aprovechas todos los recursos que te brinda este componente?**.
* Si no lo usas **¿Qué esperas para hacer la transición?** esto tiene más de un año no te quedes atrás!!.
* Aun te preguntas **¿Porque es más complicado usar un RecyclerView?** piensas que es más sencillo y conveniente usar alguna librería que facilite esta tarea.
* **¿Ya entendiste la importancia que tiene el utilizar un RecyclerView?** o aun piensas que el equipo de android solo complicó la forma de mostrar un conjunto de datos.
* **¿Como haces que tus listas se adapten de acuerdo al tamaño de pantalla del dispositivo?**


Si no sabes las respuestas en este artículo intentaremos contestarlas.

# RecyclerView

Para usar **Recyclerview-v7** recordar que hay que agregar la libreria mediante gradle `com.android.support:recyclerview-v7:+` es recomendable sustituir el símbolo "+" por el número especifico de versión.

Sabemos que un [RecyclerView](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.html) es una clase que hereda de [ViewGroup](http://developer.android.com/reference/android/view/ViewGroup.html), que al igual que [ListView](http://developer.android.com/guide/topics/ui/layout/listview.html) y [GridView](http://developer.android.com/guide/topics/ui/layout/gridview.html) nos va permitir mostrar grandes colecciones o conjuntos de datos con la única diferencia que solo se dedica a *reciclar* vistas delegando la tarea de renderizado a otros componentes que serán los que determinen la forma de acceder a los datos y cómo mostrarlos.

### LayoutManager

Es una clase estática abstracta que se encuentra dentro de la clase RecyclerView su tarea es definir el orden en que se mostrarán nuestras vistas de elementos y determinar la política de cuándo reciclar las vistas que ya no son visibles para el usuario.

### Filosofía

Ha dejado de ser una patrón por extensión recordar que las clases ListView y GridView heredan de la clase abstracta [AbsListView](http://developer.android.com/reference/android/widget/AbsListView.html) (que a su vez está hereda de la clase abstracta [AdapterView](http://developer.android.com/reference/android/widget/AdapterView.html) la cual hereda de ViewGroup.)  quien es la encargada de virtualizar una lista de elementos y delegar a sus subclases la decisión de cómo mostrar sus elementos, por esta razón un [ListView](http://developer.android.com/reference/android/widget/ListView.html) es quien decide que los elementos deben ser mostrados de forma lineal y  en forma de cuadrícula en el caso de [GridView](http://developer.android.com/reference/android/widget/GridView.html).

RecyclerView necesita del método `setLayoutManager(LayoutManager layout)` para asignar un tipo de layout que le indique cómo debe renderizar, dando origen a un patrón por composición.

Es sensato mencionar que con la llegada de **RecyclerView** y la forma en que fue diseñada su implementación se cumple un Design Principle *“Favor composition over inheritance.”* el cual tiene como objetivo dar flexibilidad.


### Tipos de LayoutManager

Existen tres implementaciones diferentes de LayoutManager que vienen incluidos dentro de la librería `Recyclerview-v7` [LinearLayoutManager](https://developer.android.com/reference/android/support/v7/widget/LinearLayoutManager.html), [GridLayoutManager](https://developer.android.com/reference/android/support/v7/widget/GridLayoutManager.html) y [StaggeredGridLayoutManager](https://developer.android.com/reference/android/support/v7/widget/StaggeredGridLayoutManager.html), tienen la habilidad de facilitar el uso de RecyclerView indicando la forma en que los elementos deben ser mostrados.


#### LinearLayoutManager

![linearLayoutManager](/content/images/2015/10/linear.png){: .center-image }

{% highlight java %}

LinearLayoutManager linearLayoutManager = new LinearLayoutManager(
            this,
            LinearLayoutManager.VERTICAL,
            false);

{% endhighlight %}

#### GridLayoutManager

![GridLayoutManager](/content/images/2015/10/grid.png){: .center-image }

{% highlight java %}

GridLayoutManager gridLayoutManager = new GridLayoutManager(
              this,
              2, //number of grid columns
              GridLayoutManager.VERTICAL,
              false);


{% endhighlight %}

#### GridLayoutManager + SpanSizeLookup

Usando el metodo `setSpanSizeLookup(SpanSizeLookup spanSizeLookup)` establecemos la cantidad de spans que debe ocupar cada elemento en el adaptador.

![CustomGridLayoutManager](/content/images/2015/10/custom_grid.png){: .center-image }

{% highlight java %}

GridLayoutManager gridLayoutManager = new GridLayoutManager(
               this,
               2, //number of grid columns
               GridLayoutManager.VERTICAL,
               false);

gridLayoutManager.setSpanSizeLookup(new GridLayoutManager.SpanSizeLookup() {
    @Override
    public int getSpanSize(int position) {
      //stagger rows custom
      return (position % 3 == 0 ? 2 : 1);
    }
});

{% endhighlight %}

#### StaggeredGridLayoutManager

![StaggeredGridLayoutManager](/content/images/2015/10/staggered.png){: .center-image }

{% highlight java %}

StaggeredGridLayoutManager staggeredGridLayoutManager = new StaggeredGridLayoutManager(
              2, //number of grid columns
              GridLayoutManager.VERTICAL);

//Sets the gap handling strategy for StaggeredGridLayoutManager
staggeredGridLayoutManager.setGapStrategy(StaggeredGridLayoutManager.GAP_HANDLING_NONE);

{% endhighlight %}


El potencial que brindan estos layouts por defecto es mas que suficiente para ser usados de forma directa en un RecyclerView, también puedes implementar un estilo de renderizado diferente a los que el framework te provee [creando tu propio LayoutManager](http://wiresareobsolete.com/2014/09/building-a-recyclerview-layoutmanager-part-1/) que se adapte a tu necesidad.


### Utiliza un tipo de LayoutManager

{% highlight java %}

RecyclerView recyclerView = new RecyclerView(this);
recyclerView.setLayoutManager(...); // sets the type of layout (LayoutManager)
setContentView(recyclerView);

{% endhighlight %}


### RecyclerView.Recycler

RecyclerView nos brinda un nuevo componente de nombre [Recycler](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.Recycler.html) una clase encargada de gestionar si una vista será (desechada o separada) `scrapped or detached` para su reutilización.

![recycler](/content/images/2015/10/recycler.png){: .center-image }

[Recordarás que un ListView o GridView](https://docs.google.com/document/d/1htu3vMmFMdMIUF-eg8JALXd9XijuR-OQ2ABxIUV42Ls/edit?usp=sharing) es quien tiene que interactuar de forma directa con el adapter para obtener las vistas que debería renderizar con RecyclerView esto ya sucede así. Esto quiere decir que un `LayoutManager` será quien interactúa de forma directa con el Recycler para obtener una nueva vista o reciclar una vista y `Recycler` será quien esté hablando con él adapter.

Si un vista es `scrapped` indica que continúa enlazada con el RecyclerView pero se ha marcado para su posible eliminación o reutilización. Cuando es reutilizada es considerada `dirty` y el adaptador es quien debe volver a enlazarla antes de ser mostrada.
Recycler implementa toda esa lógica de cuando un [Adapter](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.Adapter.html) debe crear una nueva vista o volver a enlazar datos a una ya existente y lo más importante aquí es que el LayoutManager nunca debe tocar el adapter y debe ser el `Recycler` quien esté interactuando con el.

### Filosofía

Existen dos puntos importantes dentro de recycler que le permiten trabajar de manera eficiente.

#### Scrap Heap

Es el lugar donde puedes interactuar con las vistas que no estás utilizando en el momento pero vas a volver a utilizar de forma inmediata y se consigue mediante el método
`detachAndScrapView(View child, Recycler recycler)` tener en mente que una vista `Scrapping` permitirá ser reutilizada para mostrar datos actualizados o diferentes.

#### Recyler Pool

Básicamente es donde encuentran las vistas que no necesitamos más y utilizando el método `removeAndRecycleView(View child, Recycler recycler)` removemos las vistas que no necesitamos con el diseño actual para que el método `recycleView(View view)` pueda re-enlazar y re-utilizar esas mismas vistas, cabe resaltar que siempre estarán pasando a través del adaptador para asegurar que los datos se encuentran enlazados a la vista  antes de que se devuelvan al LayoutManager.

El uso de Recycler mejora el performance ya que estamos reutilizando vistas en lugar de crear nuevas vistas cada vez que el usuario está trayendo más elementos (haciendo scroll).

### RecyclerView.Adapter

Adapter es una clase estática abstracta su principal objetivo es crear vistas para cada elemento de un RecyclerView.
La novedad radica en el uso obligatorio del patrón [ViewHolder](http://developer.android.com/training/improving-layouts/smooth-scrolling.html)  `Adapter<VH extends ViewHolder>` ya que siempre debe ser indicado en la declaración de la clase y dejar en claro que este patrón ya tiene su tiempo solo que esta vez no tenemos opción de elegir si se usa o no.

Sin embargo podrías no usar este patrón quizas e intentarlo mediante otro patrón o una combinación de patrones de diseño ;pero no creo que sea necesario ya que cumple con su objetivo. En el caso de ListView y GridView no es obligatorio el uso de `ViewHolder` pero muy recomendado para tener un buen performance y evitar el uso frecuente de `findViewById` el uso de `ViewHolder` es la pura vitamina de las listas de acuerdo con Google.

#### onCreateViewHolder

Se llama cuando RecyclerView necesita un nuevo `ViewHolder` del tipo dado para representar un elemento. `onCreateViewHolder(ViewGroup parent,int viewType)` es el encargado de inicializar y crear nuevos objetos ViewHolder para los elementos de una colección.

#### onBindViewHolder

Su tarea de este método `onBindViewHolder(ViewHolder holder, int position)` es mostrar los datos de una posición especificada, también configura el contenido de las vistas y actualiza los datos de un ViewHolder ya existente.

Existe una librería muy chula de nombre [Renderers](https://github.com/pedrovgs/Renderers) evita todo el boilerplate necesario al utilizar un RecyclerView o ListView con adaptadores, si tienes interés en usarla siempre recomiendo entender primero su funcionamiento a bajo nivel.

### RecyclerView.ItemDecoration

[ItemDecoration](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.ItemDecoration.html) una super clase que permite agregar un decorator a cada elemento o todos los elementos de un RecyclerView mediante el método `addItemDecoration(ItemDecoration decor)`, existen muchas formas para crear un ItemDecoration lo más importante es que siempre hay que heredar de `RecyclerView.ItemDecoration`.

Un ejemplo sencillo podría ser usando el método `outRect.set(int left, int top, int right, int bottom)` para insertar algo similar a un padding o margin a cada vista de nuestro RecyclerView sin necesidad de modificar los parámetros de un layout.

Pensarás que no tiene nada de importancia crear un divider cuando el ListView ya tenia por defecto su propio divider  (esa famosa línea gris). Te haz preguntado o has visto *¿Qué sucede cuando querías tener un divider diferente para algún ítem o un conjunto de ellos dentro de un ListView?* la respuesta es que se se hacían o se hacen cosas espantosas como añadir un elemento `<View … />` como parte de nuestro ItemView y era manipulado en el adapter para cumplir con los objetivos que el Product Manager requiere.

Básicamente es decirle adiós a los dividers con el uso de los decoradores ya que
proveen mucha flexibilidad permitiendo decorar un ítem o un conjunto de ellos de una manera elegante y si es importante saber que el uso de decoradores no es trivial pero sin duda un componente con mucha potencia.


* ItemDecoration para mostrar la famosa línea gris del ListView en un RecyclerView [aquí](https://github.com/erikcaffrey/RecyclerView-Examples/blob/master/app/src/main/java/androidtitlan/gdg/recyclerview_examples/widget/DividerDecoration.java).

{% highlight java %}
recyclerView.addItemDecoration(ItemDecoration decor);
{% endhighlight %}


### RecyclerView.ItemAnimator

Un clase abstracta encargada de reflejar los cambios que se hacen a un adapter, permitiendo definir las animaciones que serán visualizadas en nuestro RecyclerView al crear un ItemAnimator.

Este componente no es sencillo de implementar al igual que `ItemDecoration`, pero existe una implementación por defecto de nombre `DefaultItemAnimator` una clase que hereda de `RecyclerView.ItemAnimator` (la forma en que se crea un ItemAnimator) que permite mostrar una animación al realizar alguna acción sobre un ítem. Para asignar un ItemAnimator a un RecyclerView basta con usar el método `setItemAnimator()`.

Es importante mencionar que el feedback que se percibe al tocar un row ya no está por defecto como sucedía en ListView ;pero basta con crear un [Selector](http://developer.android.com/guide/topics/resources/drawable-resource.html) y asignarlo como background de nuestro itemView, con la llegada de material design ahora trae un efecto muy interesanté de nombre [ripple](http://android-developers.blogspot.mx/2014/10/implementing-material-design-in-your.html) que seguro ya haz trabajado con el.

{% highlight java %}
recyclerView.setItemAnimator(ItemAnimator animator);
{% endhighlight %}

### ItemTypes

En algunas ocasiones es necesario tener listas con diferentes tipos de rows con la llegada de los `ItemTypes` será más fácil realizar esta tarea, esta nueva forma de trabajar implica dejar de usar el famoso Header y Footer que se encontraban en un ListView o gridview.
Lo que han hecho es que ahora es que mediante el método `getItemViewType(int position)` para indicar que tipo de vista que debe renderizar en un RecyclerView de acuerdo a una posición validando en el `createViewHolder(ViewGroup parent, int viewType)` el tipo de vista recibido `viewType`.

### OnItemClick doesn´t exist any more

Ahora somos los encargados de crear un listener para cada elemento de una colección, generalmente se implementa en el `ViewHolder` ya que en esa clase tenemos acceso al método `getAdapterPosition()` que devolverá la posición del item asociado al ViewHolder en la colección del Adapter.
Debido a que este evento no viene más por defecto como sucedía en `ListView` usando [AdapterView.OnItemClickListener](http://developer.android.com/reference/android/widget/AdapterView.OnItemClickListener.html), esto no quiere decir que el código que se tenía en un activity o Fragment deba ser movido a un adaptador si no que debe ser delegado esta funcionalidad a nuestro ViewHolder para que se comunique con estas vistas.

### RecyclerView using Responsive grid

Los recursos son una de las herramientas más potentes que tiene Android para trabajar con UI y el uso de qualifiers permitirá aplicar diferentes configuraciones de recursos en función de densidad de pantalla, idioma, orientación, etc.
Para definir la medida que cada elemento debe ocupar en relación al tamaño de pantalla se puede conseguir de diferentes formas yo me enfocaré en dos
que son con las que he trabajado en donde GridLayoutManager juega un maginifico papel.

#### Resources & Qualifiers

Una forma fácil y sencilla para determinar el `spanCount` puede ser usando qualifiers, es tan simple como crear diferentes configuraciones tomando en cuenta el ancho mínimo , alto disponible y orientación del dispositivo .

* **values**
{% highlight xml %}
   <integer name="number_of_columns">1</integer>
{% endhighlight %}
* **values-sw600dp**
{% highlight xml %}
   <integer name="number_of_columns">2</integer>
{% endhighlight %}
* **values-sw600dp-land**
{% highlight xml %}
   <integer name="number_of_columns">3</integer>
{% endhighlight %}

{% highlight java %}

final int spans = getResources().getInteger(R.integer.number_of_columns);
GridLayoutManager gridLayoutManager = new GridLayoutManager(
              this,
              spans, //number of grid columns
              GridLayoutManager.VERTICAL,
              false);


{% endhighlight %}

Por ejemplo si es un dispositivo como un **Moto E** hará uso de la carpeta *values* mostrando solo una columna ;pero si usamos una **Nexus 9** ahora mostrará dos columnas en orientación vertical usando la carpeta *values-sw600dp* y si el dispositivo es rotado quedando una orientación horizontal se hace uso *values-sw600dp-land* para mostrar tres columnas.


#### Custom RecyclerView

Debemos crear una [CustomView](https://developer.android.com/intl/es/training/custom-views/index.html), es decir crearemos una clase que herede de `RecyclerView` y definir el tamaño de cada elemento calculando el [spanCount](https://developer.android.com/intl/es/reference/android/support/v7/widget/GridLayoutManager.SpanSizeLookup.html) automáticamente.

La forma en que funciona es que al heredar de RecyclerView estamos accediendo a su width, recordar que *GridView* tiene acceso al atributo `android:numColumns="auto_fit"` que a su vez utiliza `android:columnWidth` para calcular el número de columnas, nosotros haremos uso de este atributo para leer el valor de `android:columnWidth` mediante el constructor y asignarlo en una variable miembro `columnWidth`.

Sabemos que necesitamos establecer `columnWidth`en el método `onMeasure(int widthSpec, int heightSpec)` para calcular el `spanCount` de forma automática ;pero la aplicación podría tener un crash si esperamos hasta que se defina un `GridLayoutManager` por esta razón lo creamos en esta clase y de esta forma también accederemos a su método `setSpanCount(int spanCount)` para determinar el `spanCount`.

Puedes ver el código  de [ResponsiveRecyclerView.java](https://github.com/erikcaffrey/RecyclerView-Examples/blob/master/app/src/main/java/androidtitlan/gdg/recyclerview_examples/widget/ResponsiveRecyclerView.java).

{% highlight xml %}

<your.package.widget.ResponsiveRecyclerView
        android:id="@+id/recycler_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:columnWidth="@dimen/column_width"
        android:clipToPadding="false"/>
{% endhighlight %}


### RecyclerView new Features

Con la última versión de [Android Support Library (23.1)](https://developer.android.com/intl/es/tools/support-library/index.html?utm_campaign=android_update_important_bug_fixes_101515&utm_source=anddev&utm_medium=blog) trae algunas mejoras para RecyclerView.

`compile 'com.android.support:design:23.1.0'`

* **ItemAnimator** puede decidir mediante `canReuseUpdatedViewHolder()` si quiere volver a utilizar el mismo ViewHolder para animaciones o si RecyclerView debe crear una copia del ítem, dando como resultado que [ItemAnimator](https://developer.android.com/intl/es/reference/android/support/v7/widget/RecyclerView.ItemAnimator.html?utm_campaign=android_update_important_bug_fixes_101515&utm_source=anddev&utm_medium=blog) utilice ambos al mismo tiempo para ejecutar una animación.

* [ItemHolderInfo](https://developer.android.com/intl/es/reference/android/support/v7/widget/RecyclerView.ItemAnimator.ItemHolderInfo.html?utm_campaign=android_update_important_bug_fixes_101515&utm_source=anddev&utm_medium=blog) es otra de las aportaciones en esta versión y es definida como una estructura de datos que contiene información acerca de los enlaces de un elemento. Esta información se utiliza en el cálculo de las animaciones de los ítems.

 * Si antes heredabas de la clase ItemAnimator ahora puedes hacer uso de [SimpleItemAnimator](https://developer.android.com/intl/es/reference/android/support/v7/widget/SimpleItemAnimator.html?utm_campaign=android_update_important_bug_fixes_101515&utm_source=anddev&utm_medium=blog) un wrapper para ItemAnimator que decide si debe ejecutar mover, cambiar, añadir o eliminar una animacion.

* Ahora se provee un camino para el *backward-incompatible* de esta api y te darás cuenta de que algunos métodos se han eliminado por completo de ItemAnimator uno de ellos fue:

{% highlight java %}
recyclerView.getItemAnimator().setSupportsChangeAnimations(false)
{% endhighlight %}

Fue reemplazado por:

{% highlight java %}
ItemAnimator animator = recyclerView.getItemAnimator();
if (animator instanceof SimpleItemAnimator)
   ((SimpleItemAnimator) animator).setSupportsChangeAnimations(false);

{% endhighlight %}

### Conclusión

RecyclerView es una vista con mucho potencial que fue incorporada con el propósito de mostrar vistas de una forma sencilla y eficiente. No sólo mejora las tareas de ListView y GridView sino también sustituye a estas vistas en la mayoría de ocasiones ya que aporta mucha flexibilidad que puede suplir la funcionalidad de ambos controles, si aun lo estas dudando o quieres aprovechar al máximo cada elemento que proporciona estas en el momento indicado,  aunque la curva de aprendizaje de este control no ha sido tarea sencilla por la complejidad que tiene crear algunos de sus elementos. Solo se trata de entender cada componente que nos brinda y aprovecharlo a nuestro favor.

**"if you're afraid to change something it is clearly poorly designed. by Martin Fowler"**

### Resources

* **Android** - [Creating Lists and Cards](https://developer.android.com/training/material/lists-cards.html)
* **Dave Smith** - [Mastering RecyclerView layouts](https://www.youtube.com/watch?v=gs_C1E8HwvE)
* **Fernando Cejas & Jorge Barroso** - [Material for old school](http://es.slideshare.net/flipper83/material-old-school)
* **Chiu-Ki Chan** - [android-recyclerview repository](https://github.com/chiuki/android-recyclerview)

### Demo

#### El código está disponible

[RecyclerView-Examples en Github](https://github.com/erikcaffrey/RecyclerView-Examples)

![](/content/images/2015/10/Telecine_2015-10-18-21-56-43.gif){: .center-image }
