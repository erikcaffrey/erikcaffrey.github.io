---
layout: post
title: 'Migrate to New Android Plugin for Gradle 3.?.?'
date: '2017-12-01 17:28:10'
tags:
- android
categories:
- Android Studio
- Gradle
---

![kotlin-arch](/content/images/2017/12/gradle.png){: .center-image }

# Migrate to Android Plugin for Gradle 3.?.?

Our happy history migrating an existing android project using gradle 2.3 to new gradle version.

One of the most important announcements on the last Google I/O was [Android Studio 3](https://android-developers.googleblog.com/2017/10/android-studio-30.html) and with it the version 3 of [Android Gradle Plugin](https://developer.android.com/studio/build/gradle-plugin-3-0-0.html) which brings significant performance improvements to large multi-module projects. Unfortunately it contains some breaking changes so you must have to make some updates on your gradle configuration when switching to the new gradle plugin. This post is about sharing our experiences trying to migrate our project to new android Gradle 3.?.? plugin.

I started to migrate our android project for two main reasons: first I noticed several improvements “to large multi-module projects” and second one is that I had seen a talk [Speeding Up Your Android Gradle Builds](https://www.youtube.com/watch?v=7ll-rkLCtyk) when I attended the Google I/O this year, so 17 days ago, I took the awesome decision to start to migrating the project where I’m currently working, because I was eager to try everything I’ve already seen. The first time I tried to do the upgrade was in a small fake project, only to learn and practice, and I only spent about 30 minutes or less on it. But this article is here to tell you what happen when you want to apply the knowledge you learned in real life.

## What about the size our Android Project?

I’m working at Schibsted Media Group, on one Android project with different product flavors, so for you to understand better how was our experience, I want to give you a little description of our project:

* Hundreds of lines of code (it’s a big project)
* Directories and files for configuration, like checkstyle, Linters, internal tools to measure quality of our project, continuous integration, scripts for release process, and more.
* We use a lot of popular open source libraries
* 8 modules: features & utils modules, app module
* 8 product flavors
* We have a lot of types of tests: Unit tests, integration tests, UI tests.
* We are a lot of Android engineers: more than 30, with 10 daily active contributors

As you can see, it’s not a simple project in which making this change can take only a few minutes.

## How to start the migration?

First of all I recommend that you create a new branch (if you are using git), or create a backup, and read the documentation [Migrate to Android Plugin for Gradle 3.0.0](https://developer.android.com/studio/build/gradle-plugin-3-0-0-migration.html) before you start.

You need to add Google’s Maven repository to download the new plugin.

```gradle
repositories {
        ...
        google()
    }
```
Upgrade to the version that you need in the build.gradle file on the root of project.

**/build.gradle**

```gradle
classpath ‘com.android.tools.build:gradle:3.?.?’  
```
Are you ready to solve problems? First we need to do a build to see if our project is working like we expected after doing these changes.

```gradle
./gradlew build --info
```
## Which issues we found while doing the migration?
I’m gonna try to give you some advices to do a safe migration.

## Lint class disappear

We were using the [Lint](https://android.googlesource.com/platform/tools/build/+/143d17a4ade4f4af8f0269a4f7fc4238fcc60c68/gradle/src/main/groovy/com/android/build/gradle/tasks/Lint.groovy) class in a custom gradle task which applies to all our flavors but in *gradle 3* this class was removed, and was no changelog published about this, we discovered it when upgrading to the new Gradle plugin version.

**Remove:**

```gradle
import com.android.build.gradle.tasks.Lint
```

**Use:**

```gradle
import com.android.build.gradle.tasks.LintPerVariantTask
```

## Dependency configurations

Surely one of the most common changes which you noticed was deprecated **compile**. It might look like a trivial change, but you should be carefully when you start to modify it on your build.gradle, if you have some transitive dependencies it can be catastrophic like was for us. That’s why I would like to give you some recommendations to do it more easily.

    * [Read & understand](https://developer.android.com/studio/build/gradle-plugin-3-0-0-migration.html#new_configurations) the difference between **compile**, **compileOnly**, **api**, **runtimeOnly** & **implementation**.
    * Identify dependencies between feature modules.
    * Identify dependencies between feature modules & external dependencies.
    * Use Android Studio IDE tools like **cmd + R** to replace between different dependency configurations.
    * Identify dependencies compiled with **provided**.
    * Be carefully removing transitive dependencies. You can save a lot time understanding compilation problems

## Upgrade Kotlin

Upgrade to the last version of Kotlin. In our case it was **1.1.51**.

```gradle
ext.kotlin_version = '1.1.51'
```

## Upgrade Jacoco

We use [Jacoco](http://www.jacoco.org/jacoco/trunk/index.html) to measure java code coverage so it was great moment to update it too.
```gradle
classpath "org.jacoco:org.jacoco.core:0.7.7.201606060606"
```

## Say good bye & thanks to Gradle RetroLambda

We were using Retrolambda plugin to get java lambda support in Android but with Android Studio 3.0, not everything is about Kotlin, there are projects written in Java or Java & Kotlin mixed, as in our case. Also Android Studio support java 8 languages features in the default toolchain.

**Remove:**

```gradle
// Remove Jack Options
jackOptions {
  enabled true
  ...
}

// Remove Gradle Retrolambda

classpath ‘me.tatarka:gradle-retrolambda:3.6.0’
```

**Use:**

```gradle
compileOptions {
  sourceCompatibility JavaVersion.VERSION_1_8
  targetCompatibility JavaVersion.VERSION_1_8
}
```

If you want to get more details about this change, you can see Android Studio 3.0: [Java 8 Language Features Support](https://www.youtube.com/watch?v=LhaSi6_i2bo) video.

## Butterknife Gradle plugin issue

We use the famous Butterknife Library specifically the version **8.8.1**.

```gradle
classpath 'com.jakewharton:butterknife-gradle-plugin:8.8.1'
```

With the upgrade of Android Gradle plugin, it seems something was broken:

```gradle
Error:A problem occurred configuring project ':app'.
> Failed to notify project evaluation listener.
   > com.android.build.gradle.api.BaseVariant.getOutputs()Ljava/util/List;
```
In **ButterKnifePlugin.kt** is the offending line it calls **variant.outputs.forEach**, but the new Android Gradle 3.0 plugin requires **variants.outputs.all**.

### How should I fix it?

You have two ways to solve it:

* Use the version where it was fixed **9.0.0-SNAPSHOT**.

```gradle
classpath 'com.jakewharton:butterknife-gradle-plugin:9.0.0-SNAPSHOT'
```
Use a workaround solution:

* Downgrade using the **8.4.0 version**.

```gradle
classpath 'com.jakewharton:butterknife-gradle-plugin:8.4.0'
```

*Note: it doesn't resolve the compatibility issue with Feature Module.*

## Flavor Dimensions*

In some cases, you may want to combine configurations from multiple product flavors. For example, you may want to create different configurations for the “premium” and “free”. To do this, you should use [flavor dimensions](http://google.github.io/android-gradle-dsl/current/com.android.build.gradle.AppExtension.html#com.android.build.gradle.AppExtension:flavorDimensions%28java.lang.String[]%29).

The new plugin now requires that all flavors belong to a named flavor dimension even if you intend to use only a single dimension.

```gradle
flavorDimensions "marketplaceA", "marketplaceB"

productFlavors {
    premium {
      dimension "marketplaceA"
      ...
    }

    free {
      dimension "marketplaceA"
      ...
    }

    london {
        dimension "marketplaceB"
        ...
    }

    mexico {
        dimension "marketplaceB"
        ...
    }
}
```

if you don’t really need flavor dimensions:

```gradle
android {
    ...
    flavorDimensions "default"
}
```

## Bye BuildToolsVersions

Remove buildToolsVersions from your modules: It’s no longer needed 🎉.

## Support for legacy multidex test APK

Maybe you will found some issues about multidex on test Apk to solved it you should use **3.1.0-alpha04**.

The issue will be fixed on **android gradle plugin 3.1.0** version (more info about it: https://issuetracker.google.com/issues/37324038)

## Conclusion

The complexity of doing this migration depends a lot of the project and the tools you use. I recommend doing the changes step by step, keeping your calm to avoid spending a lot time trying to get a successful compilation. In our case, we spent hours of research. With this post I want to try saving you time. Thanks for reading and also to all my team who was involved in this funny adventure. I hope it’s useful!

#### Further Reading

* [Android Studio 3.0]()
* [About Android Gradle Plugin]()
* [Migrate to Android Plugin for Gradle 3.0.0]()
* [Speeding Up Your Android Gradle Builds]()
* [Java 8 Language Features Support]()

**"Truth can only be found in one place: the code. by Robert C. Martin"**
