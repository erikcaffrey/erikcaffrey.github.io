---
layout: post
title: 'Support of Kotlin & Architecture Components'
date: '2017-05-23 14:28:10'
tags:
- android
categories:
- Android Architecture
---

![kotlin-arch](/content/images/2017/5/kotlin-arch.png){: .center-image }

# Official support of Kotlin on Android

The last Google I/O 2017 like you know the Android team announced the [official support of Kotlin programming language](https://developer.android.com/kotlin/index.html). Kotlin is a JVM based object-oriented language that includes many ideas from functional programming. It is a brilliantly designed, mature language that will make Android development faster and as enjoyable and fun as possible.

![kotlin-annunce](https://raw.githubusercontent.com/erikcaffrey/erikcaffrey.github.io/master/content/images/2017/5/kotlin_support.jpg){: .center-image }

[Kotlin](https://kotlinlang.org/) was created by JetBrains, the team behind IntelliJ, which is the base for Android Studio. Kotlin team started the journey 6 years ago, and 2 years ago the Android community and industry around the world started to experiment with this language. Companies like Expedia, Flipboard, Pinterest or Square are using Kotlin in their production apps already. Kotlin official support by Google is a very good news and a chance to use a modern and powerful language. It's another way to create Android apps, that will help solving common headaches such as runtime exceptions (NPE) and source code verbosity. Kotlin is easy to get started and it has already been adopted by several major developers. It also plays well with the Java programming language; the effortless interoperation between the two languages has been a large part of Kotlin's appeal can be introduced gradually into existing projects, which means that your existing skills and technology investments are preserved.

## Getting started With Kotlin

The [Kotlin plugin](https://plugins.jetbrains.com/plugin/6954-kotlin) is available for the current version Android Studio. But another good news that came up at Google I/O is that now the plugin is bundled with Android Studio 3.0, and JetBrains will continue to work on the Android Studio plugin, collaborating closely with the Android Studio team.

If you are interested to learn more about the Kotlin programming language, I added some resources that can be useful.

#### Resources to start with Kotlin on Android

* [Getting started with Android and Kotlin by Jetbrains](https://kotlinlang.org/docs/tutorials/kotlin-android.html)
* [Get Started with Kotlin on Android by Google](https://developer.android.com/kotlin/get-started.html)
* [Kotlin Lang Reference](https://kotlinlang.org/docs/reference/)
* [Kotlin Blog by Jetbrains](https://blog.jetbrains.com/kotlin/)
* [Kotlin Kapt Annotation processing](https://kotlinlang.org/docs/reference/kapt.html)
* [Kotlin for Android Developers by Antonio Leiva](https://antonioleiva.com/kotlin-android-developers-book/)

# Android Architecture Components

We know that writing quality software is hard and complex, it is not only about satisfying requirements and features. For some time, the community has been exploring various architectures and patterns to create better apps. Google had nothing on these topics until one year ago when they released [Android Architecture Blue Prints](https://github.com/googlesamples/android-architecture) a collection of samples to discuss and showcase different architectural tools and patterns for Android apps created by different developers round the world.

Couple moths ago, Google Launched the [Android Architecture Components Framework](https://developer.android.com/topic/libraries/architecture/index.html). It is a set of libraries and guidelines that will help you design flexible, testable and maintainable apps, reduce boilerplate code, manage your UI components lifecycle and handle data, helping to create Android applications using separation of concerns (SoC).

## Basics about Architecture Components

* **Handling lifecycles:** Set of classes and interfaces that allow you manage the lifecycle of an activity or fragment.
* **Lifecycle:** Is a abstract class that contains the information about the lifecycle state of a component.
* **LifecycleRegistry:** Is an explicit interface allows register an object to get the lifecycle state (looks LifecycleFragment or LifecycleActivity).
* **LifecycleOwner:** An interface with a single method to know that a component has a Lifecycle.
* **Live Data:** Observable with super powers basically keeps a value and allows this value to be observed it across lifecycle changes.
* **ViewModel:** A class designed to store and manage UI-related data so that the data survives configuration changes such as screen rotations.
* **Room:** Abstraction over SQLite to allow an easy database access (like an ORM or Object Mapping for SQLite).

The libraries are available from Google's Maven repository `maven { url 'https://maven.google.com' }` only add the artifacts that you need in your project other advantage is can be used standalone.

## Kotlin Devises architecture

Kotlin Devises is a sample project used to practice Kotlin and Android Architecture Components.

### View
In progress...
```kotlin

class CurrencyFragment : LifecycleFragment() {

  companion object {
    fun newInstance() = CurrencyFragment()
  }

  private var currenciesAdapter: ArrayAdapter<String>? = null
  private var currencyViewModel: CurrencyViewModel? = null

  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    initViewModel()
    populateSpinnerAdapter()
  }

  private fun initViewModel() {
    currencyViewModel = ViewModelProviders.of(this).get(CurrencyViewModel::class.java)
  }

  private fun populateSpinnerAdapter() {
    val currencies = ArrayList<String>()
    currenciesAdapter = ArrayAdapter(activity, R.layout.item_spinner, currencies)
    currencyViewModel?.getCurrencyList()?.observe(this, Observer { currencyList ->
      currencyList!!.forEach {
        currencies.add(it.code + "  " + it.country)
      }
      currenciesAdapter!!.setDropDownViewResource(R.layout.item_spinner);
      currenciesAdapter!!.notifyDataSetChanged()
    })

  }
}

```

### ViewModel
In progress...
```kotlin

class CurrencyViewModel : ViewModel() {

  @Inject lateinit var currencyRepository: CurrencyRepository

  private var liveCurrencyData: LiveData<List<Currency>>? = null

  init {
    initializeDagger()
    loadCurrencyList()
  }

  fun getCurrencyList(): LiveData<List<Currency>>? {
    return liveCurrencyData
  }

  private fun loadCurrencyList() {
    if (liveCurrencyData == null) {
      liveCurrencyData = MutableLiveData<List<Currency>>()
      liveCurrencyData = currencyRepository.getCurrencyList()
    }
  }

  private fun initializeDagger() = CurrencyApplication.appComponent.inject(this)

}
```
### Repository
In progress...
```kotlin
class CurrencyRepository @Inject constructor(
   val roomCurrencyDataSource: RoomCurrencyDataSource,
   val remoteCurrencyDataSource: RemoteCurrencyDataSource
) : Repository {

 override fun getCurrencyList(): LiveData<List<Currency>> {
   val roomCurrencyDao = roomCurrencyDataSource.currencyDao()
   val mutableLiveData = MutableLiveData<List<Currency>>()
   roomCurrencyDao.getAllCurrencies()
       .subscribeOn(Schedulers.io())
       .observeOn(AndroidSchedulers.mainThread())
       .subscribe { currencyList ->
         mutableLiveData.value = transform(currencyList)
       }

   return mutableLiveData
 }
}
```


### Room

Room is a new powerful object Mapping for SQLite to allow easy databases access in android applications.

* Boilerplate free code
* Type Adapters ("@TypeConverters" to convert object types that aren't supported by sqlite)
* Supports LiveData objects wich means you can subscribe to receive updates.
* Support Rx Java (only can return a Flowable)
* Room understand what you are trying to do (Querys)
* Room doesn’t allow you to access to database on the main thread

### Room Components

#### Database

It's the way to define our database basically it should be an abstract class that extends `RoomDatabase` and with annotation `@Database` to define the list of entities and expose the list of data access objects.

In the sample I created `RoomsCurrencyDataSource` to storage all world currencies.

```kotlin
@Database(
    entities = arrayOf(CurrencyEntity::class),
    version = 1)
abstract class RoomCurrencyDataSource : RoomDatabase() {

  abstract fun currencyDao(): RoomCurrencyDao

  companion object {

    fun buildPersistentCurrency(context: Context): RoomCurrencyDataSource = Room.databaseBuilder(
        context.applicationContext,
        RoomCurrencyDataSource::class.java,
        "currency.db"
    ).build()
  }
}
```


#### Entity

This component represents a class that holds a database row. For each class with annotation `@Entity` a database table is created to hold the items.


```kotlin
@Entity(tableName = "currencies")
data class CurrencyEntity(
    @PrimaryKey(autoGenerate = true) val id: Long,
    var countryCode: String,
    var countryName: String
)
```

Use the annotation `@ColumnInfo` to customize the name of a field.

#### Data Access Object (DAO)

This component represents and define the contract to access on Database, should be an interface with annotation `@Dao`.


```kotlin
  @Query("SELECT COUNT(*) FROM currencies")
  fun getCurrenciesTotal(): Flowable<Int>

  @Insert
  fun insertAll(currencies: List<CurrencyEntity>)

  @Query("SELECT * FROM currencies")
  fun getAllCurrencies(): Flowable<List<CurrencyEntity>>
```


There are some already defined annotations in which you only have to create your `SQL query` like `@Query, @Insert, @Delete`.

I know there are a lot good libraries like realm that can help you with all this stuffs but SQLite was released since android 1 and works in all devices is a tested technology.

# Conclusion

In progress...

**The following diagram shows all the modules that Google recommended and how they interact with one another:**

![kotlin-arch](/content/images/2017/5/currency-arch.png){: .center-image }

### Demo
![](https://raw.githubusercontent.com/erikcaffrey/Android-Architecture-Components-Kotlin/master/art/demo.png)

#### Source Code on Github
[Android-Architecture-Components-Kotlin](https://github.com/erikcaffrey/Android-Architecture-Components-Kotlin).


#### Resources to start with Android Architecture Components

* [Android Architecture Components](https://developer.android.com/topic/libraries/architecture/index.html)
* [Adding Components to your Project](https://developer.android.com/topic/libraries/architecture/adding-components.html)
* [Android Architecture Components Samples](https://github.com/googlesamples/android-architecture-components)
* [Android Architecture Components CodeLabs](https://codelabs.developers.google.com/?cat=Android)
* [Android Conferences - Google I/O 2017](https://www.youtube.com/results?search_query=google+I%2FO+android+components)

**"Truth can only be found in one place: the code. by Robert C. Martin"**
