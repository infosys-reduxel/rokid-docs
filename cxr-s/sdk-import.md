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
Dependency Import
CXR-S SDK Package (“com.rokid.cxr:cxr-service-bridge:1.0-20260522.063600-105”).

> **Version note (2026-11-18).** `maven-metadata.xml` for `com.rokid.cxr:cxr-service-bridge` now lists release versions `1.1` through `1.8` (plus `1.7.1`–`1.7.3`), with `1.8` as the current `<release>` (`lastUpdated` 20261030075904). Rokid has not published a changelog for any of them, and this repository has only decompiled and documented `1.0`. The coordinates in this chapter are left as upstream documented them; treat anything newer as undocumented.

Add the dependency in the dependencies node of the build.gradle.kts file.

Note: The SDK requires setting minSdk ≥ 28.

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
    implementation("com.rokid.cxr:cxr-service-bridge:1.0-20260522.063600-105")
}
