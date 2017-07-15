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

The last Google I/O 2017, the Android team announced the [official support of Kotlin programming language](https://developer.android.com/kotlin/index.html). Kotlin is a JVM based object-oriented language that includes many ideas from functional programming. It is a brilliantly designed, mature language that will make Android development faster and as enjoyable and fun as possible.

![kotlin-annunce](/content/images/2017/5/kotlin-annunce.jpg){: .center-image }

[Kotlin](https://kotlinlang.org/) was created by JetBrains, the team behind IntelliJ, which is the base for Android Studio. Kotlin team started the journey 6 years ago, and 2 years ago the Android community and industry around the world started to experiment with this language. Companies like Expedia, Flipboard, Pinterest or Square are using Kotlin in their production apps already. Kotlin official support by Google is a very good news and a chance to use a modern and powerful language. It's another way to create Android apps, that will help solving common headaches such as runtime exceptions (NPE) and source code verbosity. Kotlin is easy to get started and it has already been adopted by several major developers. It also plays well with the Java programming language; the effortless interoperation between the two languages has been a large part of Kotlin's appeal can be introduced gradually into existing projects, which means that your existing skills and technology investments are preserved.

# Getting started With Kotlin

The [Kotlin plugin](https://plugins.jetbrains.com/plugin/6954-kotlin), developed by JetBrains, is available for the current version Android Studio. But another good news that came up at Google I/O is that now the plugin is bundled with Android Studio 3.0, and JetBrains will continue to work on the Android Studio plugin, collaborating closely with the Android Studio team. Android Studio 3.0 is already available -and usable- as a Preview version.

![kotlin-support](/content/images/2017/5/kotlin-support.jpg){: .center-image }

If you are interested in learning more about the Kotlin programming language to create or modify your Android apps here I added some resources that can be useful.

### Resources to start with Kotlin on Android

* [Getting started with Android and Kotlin by Jetbrains](https://kotlinlang.org/docs/tutorials/kotlin-android.html)
* [Get Started with Kotlin on Android by Google](https://developer.android.com/kotlin/get-started.html)
* [Kotlin Lang Reference](https://kotlinlang.org/docs/reference/)
* [Kotlin Blog by Jetbrains](https://blog.jetbrains.com/kotlin/)
* [Kotlin Kapt Annotation processing](https://kotlinlang.org/docs/reference/kapt.html)
* [Kotlin for Android Developers by Antonio Leiva](https://antonioleiva.com/kotlin-android-developers-book/)

# Android Architecture Components

We know that writing quality software is hard and complex, it is not only about satisfying requirements and features. For some time, the community has been exploring various architectures and patterns to create better apps. Google had nothing on these topics until one year ago when they released [Android Architecture Blue Prints](https://github.com/googlesamples/android-architecture), a collection of samples to discuss and showcase different architectural tools and patterns for Android apps created by different developers round the world.

Couple moths ago, Google Launched the [Android Architecture Components Framework](https://developer.android.com/topic/libraries/architecture/index.html). It is a set of libraries and guidelines that will help you design flexible, testable, and maintainable apps, reduce boilerplate code, manage your UI components lifecycle and handle data, helping to create Android applications using separation of concerns (SoC).

## Handling lifecycles

## Live Data

## ViewModel

## Room

## IN PROGRESS...

# Kotlin Devises architecture

The following diagram shows all the modules that Google recommended and how they interact with one another:

![kotlin-arch](/content/images/2017/5/currency-arch.png){: .center-image }

### Demo

I created an android Sample **Kotlin Devises** used to practice Kotlin and Android Architecture Components.

#### Source Code on Github

[Android-Architecture-Components-Kotlin](https://github.com/erikcaffrey/Android-Architecture-Components-Kotlin)

![](https://raw.githubusercontent.com/erikcaffrey/Android-Architecture-Components-Kotlin/master/art/demo.png)

### Resources to start with Android Architecture Components

* [Android Architecture Components](https://developer.android.com/topic/libraries/architecture/index.html)
* [Adding Components to your Project](https://developer.android.com/topic/libraries/architecture/adding-components.html)
* [Android Architecture Components Samples](https://github.com/googlesamples/android-architecture-components)
* [Android Architecture Components CodeLabs](https://codelabs.developers.google.com/?cat=Android)
* [Android Conferences - Google I/O 2017](https://www.youtube.com/results?search_query=google+I%2FO+android+components)

**"Truth can only be found in one place: the code. by Robert C. Martin"**
