---
layout: post
title: 'Escribiendo android apps con Data Binding'
date: '2015-12-16 10:19:16'
tags:
- android
categories:
- Android Architecture
---

![data-binding-android](/content/images/2015/12/mvvm.png){: .center-image }

En la actualidad como lo he mencionado en mi artículo anterior la comunidad está en el punto de madurez en donde todos o gran parte de los desarrolladores queremos hacer software [S.O.L.I.D](http://devexperto.com/principio-responsabilidad-unica/), aplicaciones que no se rompan, que funcionen, que sean mantenibles  y sobre todo testables lo que contrae a crear productos de calidad, a elevar el nivel técnico de la discusión dentro de la comunidad y sobre todo a mejorar el desarrollo de aplicaciones en este grandioso ecosistema android.

Hoy quiero incitar al debate y sobre todo explicar otra alternativa para trabajar en la capa de presentación mediante el uso de Model View ViewModel.

## Model View ViewModel

Es una derivación del patrón [Presentation Model](http://martinfowler.com/eaaDev/PresentationModel.html) descrito por el gran Martin Fowler y desarrollador por Ken Cooper and Ted Peters dos arquitectos de software de Microsoft  quienes convirtieron este patrón en la base del desarrollo de apliciones dentro de esta famosa compañia.

MVVM se enfoca en abstraer la implementación concreta de la vista, es decir su comportamiento y estado, lo que nos permitirá separar las vistas de nuestra lógica de negocio y capa de datos. Básicamente la forma en que se logra es mediante un ViewModel que permite exponer los objetos de datos de un modelo y maneja la lógica de cómo se tienen que pintar nuestros datos dentro de las vistas.

![mvvm](/content/images/2015/12/mvvm_flow.png){: .center-image }

Al igual que **MVC** y [MVP](https://erikcaffrey.github.io/ANDROID-mvp/) el **MVVM** se compone de tres componentes principales:

#### Model

Es el encargado de encapsular y mantener la lógica de negocio y validación de nuestra aplicación.

#### View

La vista es la responsable de definir la estructura, el diseño y la apariencia de lo que el usuario ve en la pantalla en android se consigue mediante un conjunto layouts en xml normalmente. Es importante mencionar que una vista puede tener su propio *ViewModel*.

#### ViewModel

Es el encargado de actuar como un enlace entre la vista y el modelo pero también se encarga de mantener la lógica de presentación (Tomará las decisiones de cómo se tienen que pintar nuestros datos en nuestra UI ) y el estado de la vista para ser capaz de notificar y estar en contacto con todos los elementos o propiedades de nuestras vistas que van a ser reactivas.


### ¿Cual es la ventaja de usar MVVM?

En este momento estarás pensando que MVVM es igual MVP por que solo hemos cambiando un **Presenter** por un **ViewModel**  pero la respuesta es que no pero algo que tienen en comun es que sí proveen las mismas ventajas ambos patrones como lo es la separación de las vistas de la lógica de negocio y la capa de datos al igual que escribir diferentes tipos de test solo que *MVVM* lo hace de una forma muy distinta ya que hace uso de un [Data binding engine](https://en.wikipedia.org/wiki/Data_binding) (revisa la documentación para que entiendas un mejor concepto de cómo funciona esta libreria) el cual le permite a nuestras vistas ser reactivas, es decir ser notificadas por el **ViewModel** atravez de este engine de binging o crear un mecanismo que simule ese compartamiento.

En android este patrón está ganando terreno con la llegada de [Data Binding Library](http://developer.android.com/intl/es/tools/data-binding/guide.html) la libreria que anunció google en el I/O 2015 que permite escribir aplicaciones android usando este principio y dando origen a una programación declarativa.


## People-MVVM

Es un ejemplo de Model View View Model que lo cree con la intención de explicar cómo funciona este patrón dentro de nuestras aplicaciones android.

También lo utilice para una plática que di hace unos dias en el  [Meetup Androidinights](http://www.meetup.com/es/Androidinights/) que organizó en la Ciudad de México, aquí puedes encontrar los [slides](https://speakerdeck.com/erikcaffrey/mvvm-android) y por supuesto el código esta disponible esta en la parte final del post.

![people](/content/images/2015/12/people.png){: .center-image }

Para dar un pequeño contexto y entender cómo funciona el data binding explicare brevemente únicamente como funciona el [item_people.xml](https://github.com/erikcaffrey/People-MVVM/blob/master/app/src/main/res/layout/item_people.xml) de la lista recuerda que en el código podrás verlo más de cerca.

Cada instancia **People** se muestra mediante una vista dentro del Recyclerview.

![item_people](/content/images/2015/12/item_people.png){: .center-image }

#### Model

Muy simple modelo consiste en la lógica de negocio que corresponde a un ítem en este caso el modelo se llama [People](https://github.com/erikcaffrey/People-MVVM/blob/master/app/src/main/java/com/example/jhordan/people_mvvm/model/People.java)  y se representa de la siguiente forma.

{% highlight java %}

public class People implements Serializable {

  @SerializedName("gender") public String mGender;

  @SerializedName("name") public Name mName;

  @SerializedName("location") public Location mLocation;

  @SerializedName("email") public String mMail;

  @SerializedName("login") public Login mUserName;

  @SerializedName("phone") public String mPhone;

  @SerializedName("cell") public String mCell;

  @SerializedName("picture") public Picture mPicture;

  public String mFullName;

}



{% endhighlight %}

#### View

Nuestra vista es la responsable de definir el diseño, la apariencia y la estructura de sus componentes idealmente debe ser construida por completo de XML. La vista recupera sus datos de un ViewModel a través del uso de un Binding  luego, en tiempo de ejecución, el contenido de la interfaz de usuario puede actualizarse cuando las propiedades del ViewModel son notificadas que hubo un cambio.


{% highlight xml %}

<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools">

    <data>

        <variable
            name="peopleViewModel"
            type="com.example.jhordan.people_mvvm.viewmodel.ItemPeopleViewModel"/>
    </data>


    <RelativeLayout
        android:id="@+id/item_people"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="?attr/selectableItemBackground"
        android:onClick="@{peopleViewModel.onItemClick}"
        android:padding="@dimen/spacing_large">


        <de.hdodenhof.circleimageview.CircleImageView
            android:id="@+id/image_people"
            android:layout_width="80dp"
            android:layout_height="80dp"
            app:imageUrl="@{peopleViewModel.pictureProfile}"
            app:border_color="@color/white"/>


        <TextView
            android:id="@+id/label_name"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_alignTop="@+id/image_people"
            android:layout_marginStart="@dimen/spacing_large"
            android:layout_marginTop="@dimen/spacing_large"
            android:layout_toEndOf="@+id/image_people"
            android:text="@{peopleViewModel.fullName}"
            android:textColor="@android:color/primary_text_light"
            android:textSize="16sp"
            android:textStyle="bold"
            tools:text="mr gary allen"/>

        <TextView
            android:id="@+id/label_phone"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_alignStart="@+id/label_name"
            android:layout_below="@+id/label_name"
            android:text="@{peopleViewModel.cell}"
            android:textColor="@android:color/secondary_text_light"
            android:textSize="14sp"
            tools:text="0729-256-147"/>

        <TextView
            android:id="@+id/label_mail"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_alignStart="@+id/label_phone"
            android:layout_below="@+id/label_phone"
            android:text="@{peopleViewModel.mail}"
            android:textColor="@android:color/secondary_text_light"
            android:textSize="14sp"
            tools:text="gary.allen@example.com"/>

    </RelativeLayout>


</layout>

{% endhighlight %}


El tag **layout** indica que nuestro xml será una vista bindable o hará uso del data binding para que al pasar por el Layout Processor sea transformado a un xml que versiones anteriores puedan entender. Al hacer esto creará una clase Binding que es la que contendrá la referencia a nuestras vistas puedes ver un ejemplo de esa clase generada atravez de un layout bindable aquí [PeopleAdapter](https://github.com/erikcaffrey/People-MVVM/blob/master/app/src/main/java/com/example/jhordan/people_mvvm/PeopleAdapter.java).

Otro aspecto importante es el tag *data* que es usado para especificar el ViewModel del cual estaremos trayendo datos del modelo y representandolos en la vista.

#### ViewModel

Es el componente que actúa como enlace entre la vista y el modelo, también es el responsable de toda la lógica que indica cómo se debe pintar a nuestra vista al igual se encarga de acceder a los métodos y propiedades del modelo, que luego se pondrán a disposición de la vista.

Cada método es una propiedad de nuestro modelo que vamos a pintar en nuestra vista [dale una revisada a cómo se deben llamar cada propiedad o metodo dentro de nuestra vista o layout](http://developer.android.com/intl/es/tools/data-binding/guide.html).

{% highlight java %}

public class ItemPeopleViewModel extends BaseObservable {

    private People mPeople;
    private Context mContext;

    public ItemPeopleViewModel(People people , Context context) {
        mPeople = people;
        mContext = context;
    }

    public String getFullName() {
        mPeople.mFullName = mPeople.mName.mTitle + "." + mPeople.mName.mFirts + " " + mPeople.mName.mLast;
        return mPeople.mFullName;
    }

    public String getCell(){
        return mPeople.mCell;
    }

    public String getMail(){
        return mPeople.mMail;
    }

    public String getPictureProfile(){
        return mPeople.mPicture.large;
    }

    @BindingAdapter({"bind:imageUrl"})
    public static void loadImage(ImageView view, String imageUrl) {
        Glide.with(view.getContext()).load(imageUrl).into(view);
    }

    public void onItemClick(View view){
        mContext.startActivity(PeopleDetailActivity.launchDetail(view.getContext(),mPeople));
    }
    public void setPeople(People people) {
        mPeople = people;
        notifyChange();
    }


}


{% endhighlight %}


### Conclusión

El MVVM suele ser muy potente con la ayuda de un data binding engine su uso está cambiando realmente la manera en la que desarrollamos aplicaciones haciendo una programación más declarativa lo que debe quedar muy claro es que usar MVP o MVVM tienden hacia la misma finalidad (aislar la implementación concreta de modelo y la vista) pero con implementaciones distintas y ambos mejoran la mantenibilidad, testabilidad y reusabilidad de código. Si algo podría decir es que para mi el uso de DataBinding (ahorra mucho código repetitivo así como el uso de librerías externas para bindear vistas) permite que nuestras activities, fragments o lo que sea sean más limpias claras y concisas ya que normalmente siempre es la parte más castigada de nuestras aplicaciones.

Recuerda que lo mejor siempre es usar el que mejor entiendas ya que con ambos puedes obtener la misma finalidad.

He dado una pequeña charla hablando sobre estos temas que puedes ver aquí [GDG Open Lima Hangout #6 - El Android del Pasado, Presente / Futuro](https://www.youtube.com/watch?v=_e7aACEAfv4).

**"The Future of Programming"** [Bret Victor](https://www.youtube.com/watch?v=8pTEmbeENF4)

### Resources
* **Microsoft** - [The MVVM Pattern](https://msdn.microsoft.com/en-us/library/hh848246.aspx)
* **George Mount & Yigit Boyar** - [Data Binding -- Write Apps Faster](https://www.youtube.com/watch?v=NBbeQMOcnZ0)
* **Github Example by hitherejoe** - [MVVM_Hacker_News](https://github.com/hitherejoe/MVVM_Hacker_News)
* **Android documentation** [Data Binding Guide](https://developer.android.com/intl/es/tools/data-binding/guide.html)

### Demo

#### El código está disponible

[People-MVVM en Github](https://github.com/erikcaffrey/People-MVVM)

![](/content/images/2015/12/Telecine_2016-03-15-23-23-27.gif)
