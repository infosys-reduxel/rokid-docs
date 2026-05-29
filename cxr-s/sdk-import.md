SDK Import
This chapter takes the use of Kotlin DSL (build.gradle.kts) as an example

Configure Maven Repository
The CXR-S SDK utilizes Maven for online management of SDK packages.

Maven repository address: (“https://maven.rokid.com/repository/maven-public/”)

Locate the settings.gradle.kts file and add the Maven repository to the repositories section within the dependencyResolutionManagement node.



pluginManagement {
    repositories {
        google {
            content {
                includeGroupByRegex("com\\.android.*")
                includeGroupByRegex("com\\.google.*")
                includeGroupByRegex("androidx.*")
            }
        }
        mavenCentral()
        gradlePluginPortal()
    }
}
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        maven {
            url = uri("https://maven.rokid.com/repository/maven-public/")
        }
        mavenCentral()
    }
}
 
rootProject.name = "CXRServiceDemo"
include(":app")
## Dependency Import

CXR-S SDK Package. The current Maven release is **`com.rokid.cxr:cxr-service-bridge:1.0`** (snapshot fingerprint `1.0-20260212.103714-88`, metadata last updated 2026-05-22).

Add the dependency in the `dependencies` node of the `build.gradle.kts` file.

Note: The SDK requires setting `minSdk` ≥ 28.

```kotlin
//...Other Settings
android {
    //...Other Settings
    defaultConfig {
        //...Other Settings
        minSdk = 28
    }
    //...Other Settings
}
dependencies {
    //...Other Settings
    implementation(“com.rokid.cxr:cxr-service-bridge:1.0-20260212.103714-88”)
}
```
