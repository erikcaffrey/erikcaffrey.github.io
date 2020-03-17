---
layout: post
cover:  assets/images/2020/3/launch-workflow.png
title: Distribuye una Android App (Part 2)
date: 2020-03-16 00:00:00 +0545
categories: blog
author: erik
---

# WORK IN PROGRESS!

Antes de comenzar con la integración técnica te sugiero leer el [post anterior](https://erikjhordan-rey.github.io/blog/2020/03/15/ANDROID-automate-deploy-and-test-android-apps.html) para entender el **workflow** que estaremos implementando. 

## Sign your app automatically

Sabemos que android requiere que todos los APK o Bundle estén firmados digitalmente con un certificado antes de que se instalen en un dispositivo o se actualicen. Firmar una aplicación se puede hacer de distintas formas ;pero para poder integrar con nuestro servicio de integración continua y este pueda generar un build de forma automática necesitamos que la configuración local del proyecto sea compatible con Travis CI. 

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

    signingProduction = {
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

En la función **signingDebug** y **signingProduction** sirven para asignar el **keyAlias**, **storePassword**, **keyPassword** y todo lo necesario para [firmar un build](https://developer.android.com/studio/publish/app-signing#generate-key). Puedes mirar que hay una validación **propertiesFile.exists()** encargada de validar si está ejecutando en local (AS) o en un Travis Job, ya que va y busca la configuración en local en el archivo `debug.properties` y `release.properties`(Estos archivos estan normalmente en el .gitignore), si no encuentra lee las [variables de entorno declaradas](https://docs.travis-ci.com/user/environment-variables/) en Travis CI. 

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

Una de las tareas que en ocasiones se vuelve tediosa de mantener es el número y nombre de una versión, cada que hacemos un release nuevo implica modificar esto de forma manual para evitarlo usualmente creamos un archivo **version.properties** el cual contiene el número de versión de la aplicación y un script **app-distribution.gradle** cuyo objetivo es leer el archivo properties y generar el nombre de la versión, ambos archivos estan ubicados en **/AndroidStudio/android-project/gradle-scripts/distribution**. 

**version.properties**

Cada que realices un release solo tienes que incrementar alguno de estos valores. 

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

Para usar el script gradle que hemos generado se aplica en el **build.gradle** de el modulo`app`.

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

Usa la variables **version_code** y **version_name** que declaradas en defaultConfig. 

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