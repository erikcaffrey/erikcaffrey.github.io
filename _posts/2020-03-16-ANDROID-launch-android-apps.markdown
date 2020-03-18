---
layout: post
cover:  assets/images/2020/3/launch-workflow.png
title: Distribuye una Android App (Part 2)
date: 2020-03-16 00:00:00 +0545
categories: blog
featured: true
author: erik
---

Más que mostrar toda una integración técnica paso a paso, mi objetivo principal es compartirte algunos tips y recomendaciones de mi experiencia haciendo este tipo de implementaciones, de igual forma me gustaría recomendarte pasar por mi [artículo anterior](https://erikjhordan-rey.github.io/blog/2020/03/15/ANDROID-automate-deploy-and-test-android-apps.html) para tener una visión general sobre el tema.

## Sign your app automatically

Sabemos que android requiere que todos los APK o Bundle estén firmados digitalmente con un certificado antes de que se instalen en un dispositivo o se actualicen. Firmar una aplicación se puede hacer de distintas formas; pero para poder integrar con nuestro servicio de integración continua y éste pueda generar un build de forma automática necesitamos que la configuración local del proyecto sea compatible con Travis CI. 

Usualmente lo resuelvo creando un archivo **signing.gradle** ubicado en **/AndroidStudio/android-project/gradle-scripts/google**.

```gradle

ext {

    signingDebug = {
        def propertiesFile = loadPropertiesFile("debug")
        android.signingConfigs.debug {
            storeFile loadFileFromGoogleDir("debug.keystore")
            if (propertiesFile.exists()) {
                def properties = loadProperties(propertiesFile)
                storePassword properties.APP_DEBUG_STORE_PASSWORD
                keyAlias properties.APP_DEBUG_KEY_ALIAS
                keyPassword properties.APP_DEBUG_KEY_PASSWORD
            } else {
                storePassword System.getenv("APP_DEBUG_STORE_PASSWORD")
                keyAlias System.getenv("APP_DEBUG_KEY_ALIAS")
                keyPassword System.getenv("APP_DEBUG_KEY_PASSWORD")
            }
        }
    }

    signingProd = {
        def propertiesFile = loadPropertiesFile("release")
        android.signingConfigs.prod {
            storeFile loadFileFromGoogleDir("release.keystore")
            if (propertiesFile.exists()) {
                def properties = loadProperties(propertiesFile)
                storePassword properties.APP_STORE_PASSWORD
                keyAlias properties.APP_KEY_ALIAS
                keyPassword properties.APP_KEY_PASSWORD
            } else {
                storePassword System.getenv("APP_STORE_PASSWORD")
                keyAlias System.getenv("APP_KEY_ALIAS")
                keyPassword System.getenv("APP_KEY_PASSWORD")
            }
        }
    }

    loadPropertiesFile = { name ->
        return loadFileFromGoogleDir("${name}.properties")
    }

    loadProperties = { propertiesFile ->
        def properties = new Properties()
        properties.load(new FileInputStream(propertiesFile))
        return properties
    }

    loadFileFromGoogleDir = { fileName ->
        def googleDirPath = "gradle-scripts/google/"
        rootProject.file("${googleDirPath}${fileName}")
    }
}

```

En la función **signingDebug** y **signingProduction** sirven para asignar el **keyAlias**, **storePassword**, **keyPassword** y todo lo necesario para [firmar un build](https://developer.android.com/studio/publish/app-signing#generate-key). Puedes mirar que hay una validación **propertiesFile.exists()** encargada de validar si está ejecutando en local (AS) o en un Travis Job, ya que busca la configuración en local en el archivo `debug.properties` y `release.properties`(Estos archivos están normalmente en el .gitignore) y si no encuentra lee las [variables de entorno declaradas](https://docs.travis-ci.com/user/environment-variables/) en Travis CI. 

`debug.properties`

```gradle 
APP_DEBUG_STORE_PASSWORD=YourStorePassword
APP_DEBUG_KEY_ALIAS=YourAlias
APP_DEBUG_KEY_PASSWORD=YourkeyPassword
```

Para usar el script gradle que hemos generado se aplica en el  **build.gradle** de `app`.

```gradle

apply plugin: 'com.android.application'
//...
apply from: rootProject.file("gradle-scripts/google/signing.gradle")


signingConfigs {

        debug {
            signingDebug()
        }

        prod {
            signingProd()
        }
    }

```

## Automate versionCode and versionName

Una de las tareas que en ocasiones se vuelve tediosa de mantener es el número y nombre de una versión, cada que hacemos un release nuevo implica modificar esto de forma manual para evitarlo usualmente creamos un archivo **version.properties** el cual contiene el número de versión de la aplicación y un script **app-distribution.gradle** cuyo objetivo es leer el archivo properties y generar el nombre de la versión, ambos archivos están ubicados en **/AndroidStudio/android-project/gradle-scripts/distribution**. 

**version.properties**

Cada que realices un release sólo tienes que incrementar alguno de estos valores. 

```gradle
MAJOR=1
MINOR=0
PATCH=0
```

**app-distribution.gradle**

```gradle

import java.text.SimpleDateFormat

ext {
    versionMajor = getVersionMajor()
    versionMinor = getVersionMinor()
    versionPatch = getVersionPatch()
    version_name = "${versionMajor}.${versionMinor}.${versionPatch}"
    version_code = versionMajor * 10000 + versionMinor * 100 + versionPatch
    version_date = generateBuildTime()
}

private def getVersionMajor() {
    Properties properties = getVersionProperties()
    return properties['MAJOR'].toInteger()
}

private def getVersionMinor() {
    Properties properties = getVersionProperties()
    return properties['MINOR'].toInteger()
}

private def getVersionPatch() {
    Properties properties = getVersionProperties()
    return properties['PATCH'].toInteger()
}

private def getVersionProperties() {
    def content = rootProject.file("gradle-scripts/distribution/version.properties")
    Properties properties = new Properties()
    InputStream is = new ByteArrayInputStream(content.getBytes())
    properties.load(is)
    return properties
}

private static def generateBuildTime() {
    return new SimpleDateFormat("dd/MM/yyyy - hh:mm aa").format(new Date())
}

```

Para usar el script gradle que hemos generado se aplica en **build.gradle** del módulo `app`.

```gradle 

apply plugin: 'com.android.application'
//...
apply from: rootProject.file("gradle-scripts/distribution/app-distribution.gradle")

```

Si utilizas **multi-module** debes proveer este script en todos los módulos del proyecto o utilizar `subprojects`.

```gradle 

subprojects {

    apply from: rootProject.file("gradle-scripts/distribution/app-distribution.gradle")
}

```

Usa las variables **version_code** y **version_name** en defaultConfig. 

```gradle

 defaultConfig {
        applicationId ...
        minSdkVersion ...
        targetSdkVersion ...
        versionCode version_code
        versionName version_name
    }

```

## Firebase App Distribution + Travis CI

Para comenzar la integración mediante gradle, es necesario agregar el plugin de `firebase-appdistribution-gradle` en `build.gradle` del proyecto.

```gradle

buildscript {
    repositories {
        google()
    }
    dependencies {
        classpath 'com.google.firebase:firebase-appdistribution-gradle:1.3.1'
    }
}

```

Aplicar el plugin en el **build.gradle** del módulo `app`.

```gradle

apply plugin: 'com.android.application'
// ...
apply plugin: 'com.google.firebase.appdistribution'


```

###  Configure Firebase App Distribution

Para distribuir una aplicación en firebase es necesario configurar [firebaseAppDistribution](https://firebase.google.com/docs/app-distribution/android/distribute-gradle).

```gradle
       firebaseAppDistribution {
                releaseNotesFile = "path/release-notes.txt"
                groups = "internal-testers"
                serviceCredentialsFile = rootProject.file("gradle-scripts/distribution/service-account.json")
        }
```

### App Distribution Build Parameters

En este [link](https://firebase.google.com/docs/app-distribution/android/distribute-gradle) puedes encontrar lista completa de parámetros.

**releaseNotesFile:** Se usa para especificar el archivo `.txt` que contendrá nuestro release notes.

**groups:** Es el grupo de testers para el que va dirigido el build, podrías utilizar otro parámetro llamado `testers` aúnque implica ingresar una lista de correos, así que te recomiendo crear un grupo de testers en la consola de firebase y de esa forma controlar quienes tienen acceso a tus builds.

**serviceCredentialFile:** Es necesario especificar un [service account](https://console.cloud.google.com/projectselector2/iam-admin/serviceaccounts) ya que es la forma en que hacemos la autenticación para poder deployar una aplicación en firebase y Google Play desde nuestro sistema de CI.

**apkPath:** Se utiliza para especificar a donde se tiene que buscar el APK generado, si no se especifica buscará en el default **build/outputs/apk/debug/app-yourbuildtype.apk**.

### BuildTypes + FirebaseAppDistribution

#### Debug Build Type

En el [flujo que definimos](https://erikjhordan-rey.github.io/blog/2020/03/15/ANDROID-automate-deploy-and-test-android-apps.html) cada merge a **master** tiene que distribuir un build en firebase con el último commit al termino de un Job en travis CI. 

```gradle

        debug {
            def type = "debug"
            applicationIdSuffix ".${type}"
            versionNameSuffix "-${type}"
            ....
            signingConfig signingConfigs.debug

            firebaseAppDistribution {
                releaseNotesFile = "./release-notes.txt"
                groups = "internal-testers"
                serviceCredentialsFile = rootProject.file("gradle-scripts/distribution/service-account.json")
            }
        }

```

Para generar el release notes especificado en **releaseNotesFile** se hace mediante la task **generateReleaseNotes**, se encarga de crear el archivo `./release-notes.txt` con la información del commit del último merge a **master**.

```gradle

 task generateReleaseNotes {
   doLast({
       def releaseNotes = new File('release-notes.txt')
       releaseNotes.delete()
       releaseNotes << "Release Notes - Version $name_version (${date_version})\n\n"
       releaseNotes << "Change Log \n\n"
       def lastCommitCommand = "git log --format=%s --no-merges -n 1"
       def commitMessageLines = lastCommitCommand.execute()
       commitMessageLines.in.eachLine { line -> releaseNotes << "* " + line + "\n\n\n" }
       releaseNotes << "This file was automatically generated."
   })
}
``` 

El archivo generado contiene la siguiente información:

**./release-notes.txt**

```gradle 
Release Notes - Version 1.0.0 (16/03/2020 - 01:00 PM)

Change Log 

* Included awesome feature 

This file was automatically generated.
```

#### Release Build Type

Para el build de producción la configuración es realmente idéntica a diferencia que no generamos el release notes automáticamente, ya que la idea es que cuando creamos un release candidate lo modifiquemos con los textos necesarios y amigables dado que son los que se mostrarán en Google Play Store. 

```gradle 

        release {
            ...
            firebaseAppDistribution {
                releaseNotesFile = rootProject.file("gradle-scripts/distribution/release_notes.txt")
                groups = "internal-testers"
                serviceCredentialsFile = rootProject.file("gradle-scripts/distribution/service-account.json")
            }
        }

```

### Configure Travis Job

Una vez que tenemos la configuración en el proyecto android es posible realizar la integración con [Travis CI + Android](https://docs.travis-ci.com/user/languages/android/). Cada Job en Travis CI tiene un [ciclo de vida](https://docs.travis-ci.com/user/job-lifecycle/) y la parte que necesitamos enteder es conocida como `deploy`, la cual es opcional dentro de la configuración del script, es decir no la necesitas para poder correr tu proyecto en Travis CI, sólo se agrega esta fase si se necesita automatizar un deploy. 

#### Debug Release

Como he mencionado antes en nuestro [workflow](https://erikjhordan-rey.github.io/blog/2020/03/15/ANDROID-automate-deploy-and-test-android-apps.html), cada merge a master debe hacer deploy automáticamente en firebase app distribution y para lograrlo es necesario especificarle al job de travis el branch de donde debe generar el build y el conjunto de tasks a ejecutar. 

```console 

deploy:
  - provider: script
    script:
      ./gradlew generateReleaseNotes assembleDebug appDistributionUploadDebug
    on:
      branch: master
    skip_cleanup: true

```

#### Prod Release

Para generar un build de release y distribuirlo en firebase solemos definir una condición `$TRAVIS_BRANCH =~ ^release-.*$` que lo que hace es validar que nuestro branch contenga un nombre como **release-1.0** para poder generar ejecutar el script. Es importante mencionar que este branch es generado a partir de **master** cuando el equipo decide hacer un release por lo que a partir de ese momento se convierte en un branch candidate. 

```console 

  - provider: script
    script:
      ./gradlew assembleRelease appDistributionUploadRelease
    on:
      all_branches: true
      condition: $TRAVIS_BRANCH =~ ^release-.*$
    skip_cleanup: true

```

### App Bundle in Firebase App Distribution

La aplicación en la que trabajo actualmente está diseñada para hacer uso de [dynamic features](https://developer.android.com/guide/app-bundle/dynamic-delivery) por lo que en Google Play solemos distribuir un [app bundle](https://developer.android.com/guide/app-bundle) ;pero desafortunadamente **aún no** es posible subir un file `.aab` en firebase app distribution, no es un formato soportado, así para darle la vuelta generamos un **apk universal** mediante las tasks`packageDebugUniversalApk` o `packageReleaseUniversalApk` evitando instalar el [bundletool](https://github.com/google/bundletool). Es una solución que nos permite hacer pruebas internas con un build de release de forma fácil sin preocuparnos del todo en este momento por cómo probar un app bundle, ya que ahora mismo la única forma de hacerlo es mediante Google Play. 

La configuración utilizada es la siguiente: 

Es importante entender que el output de ejecutar estas tareas cambia el path default por lo que es necesario utilizar el parámetro `apkPath` para especificar el lugar a donde debe ir a buscar el `apk` que normalmente es en el path **app/build/outputs/universal_apk/release/app-release-universal.apk** para el caso de release. 

**build.gradle**

```gradle

     firebaseAppDistribution {
                releaseNotesFile = rootProject.file("gradle-scripts/distribution/release_notes.txt")
                groups = "internal-testers"
                apkPath = rootProject.file("app/build/outputs/universal_apk/release/app-release-universal.apk")
                serviceCredentialsFile = rootProject.file("gradle-scripts/distribution/service-account.json")
     }

```

**.travis.yml**

```console

script:
  - ./gradlew packageReleaseUniversalApk

deploy:
  
  - provider: script
    script:
      ./gradlew appDistributionUploadRelease
    on:
      all_branches: true
      condition: $TRAVIS_BRANCH =~ ^release-.*$
    skip_cleanup: true

```

## Google Play + Travis CI

Generar un release a Google Play Store es medianamente simple si utilizas el conocido plugin [gradle-play-publisher](https://github.com/Triple-T/gradle-play-publisher), la configuración es sencilla y se integra de una forma rápida con Travis CI. En nuestro caso lo que hacemos es liberar un **app bundle** en nuestro stage “internal” cuando se crea un **tag** release `1.0.0`. Así que puedes iniciar pruebas con usuarios reales e ir pasando de stage hasta llegar al 100% en producción. 

**build.gradle**

```gradle

classpath "com.github.triplet.gradle:play-publisher:2.7.2"

...

apply plugin: "com.github.triplet.play"

...

play {
    serviceAccountCredentials = rootProject.file("gradle-scripts/distribution/service-account.json")
    track = "internal"
    userFraction = 0.5
    releaseStatus = "inProgress"
}


```

**.travis.yml**

```console

 deploy:

  - provider: script
    script:
      ./gradlew publishReleaseBundle
    on:
      tags: true
    skip_cleanup: true

```

### Android + Travis CI 

Desde un enfoque más general un archivo `.travis.yml` tiene la siguiente estructura:

```console

language: android
jdk: oraclejdk8

branches:
 only:
  - master
  # Release Branches : "release-0.1"
  - /^release-\d+\.\d+$/
  # Release Tags: "0.1.0"
  - /^\d+\.\d+\.\d+$/

before_cache:
  - rm -f  $HOME/.gradle/caches/modules-2/modules-2.lock
  - rm -fr $HOME/.gradle/caches/*/plugin-resolution/

cache:
  directories:
    - $HOME/.gradle/caches/
    - $HOME/.gradle/wrapper/
    - $HOME/.android/build-cache

env:
  global:
    - ADB_INSTALL_TIMEOUT=8
    - ANDROID_API_LEVEL=28
    - ANDROID_BUILD_TOOLS_VERSION=28.0.3

android:
  components:
    - tools
    - platform-tools
    - build-tools-$ANDROID_BUILD_TOOLS_VERSION
    - android-$ANDROID_API_LEVEL
    - extra-android-support
    - extra-google-google_play_services
    - extra-google-m2repository
    - extra-android-m2repository

licenses:
  - 'android-sdk-preview-license-.+'
  - 'android-sdk-license-.+'
  - 'google-gdk-license-.+'

script:
  #execute checkstyles, tests suite, linters, code analysis.
  - ./gradlew packageDebugUniversalApk packageReleaseUniversalApk

deploy:
  - provider: script
    script:
      ./gradlew generateReleaseNotes appDistributionUploadDebug
    on:
      branch: master
    skip_cleanup: true

  - provider: script
    script:
      ./gradlew appDistributionUploadRelease
    on:
      all_branches: true
      condition: $TRAVIS_BRANCH =~ ^release-.*$
    skip_cleanup: true

  - provider: script
    script:
      ./gradlew publishReleaseBundle
    on:
      tags: true
    skip_cleanup: true
```

Esta es una configuración que podría ayudarte de manera inicial aunque puedes mejorarla en cuanto a tiempos de build, conectar con una granja de dispositivos para ejecutar tus tests o a usar algún TestLab, meter más checkers son cosas que puedes ir agregando y mejorando a medida que tu proyecto va creciendo en conjunto con el número de contribuidores.

### Conclusion

La importancia de mantener un producto de software en estado liberable, en el cual obtener feedback sea simple, donde podamos responder a cualquier falla e iterar de forma rápida es una disciplina en la que se debe trabajar fuertemente como equipo para lograrlo. Asegurar la calidad de cada cosa que llega a producción, así como proveer mejores experiencias a nuestros usuarios son tareas que a veces como olvidamos; pero recuerda nuestra misión es que el usuario disfrute y se ayude del producto que desarrollamos y automatizar es un gran paso para frenar todas esas fricciones manuales que vivimos en el día a día, es mejorar los procesos internos dentro de una compañía, es hacer nuestro trabajo de forma más profesional que contribuye y genera valor para todo el equipo. 

[Leer Parte 1](https://erikjhordan-rey.github.io/blog/2020/03/15/ANDROID-automate-deploy-and-test-android-apps.html)

**¡Espero que sea de ayuda y si lo encuentras interesante ayúdame a compartirlo!**

#### Further reading

* [Build Script Basics](https://docs.gradle.org/current/userguide/tutorial_using_tasks.html)
* [Building an Android Project on Travis CI](https://docs.travis-ci.com/user/languages/android/)
* [Firebase App Distribution](https://firebase.google.com/products/app-distribution)
* [Gradle Play Publisher](https://github.com/Triple-T/gradle-play-publisher)
* [BASH Programming - Introduction HOW-TO](https://tldp.org/HOWTO/Bash-Prog-Intro-HOWTO.html)

**Redundant comments are just places to collect lies and misinformation. by Robert C. Martin**
