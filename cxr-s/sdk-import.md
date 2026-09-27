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
CXR-S SDK Package (“com.rokid.cxr:cxr-service-bridge:1.4”).

> **Version note:** as of 2026-09-26, Maven's `<release>` for `cxr-service-bridge` is `1.4` (previously `1.0`, or the timestamped snapshot build `1.0-20260522.063600-105` this doc cited before this update). There is no official Rokid changelog for this artifact — see [release-notes.md](release-notes.md) for a provisional binary-diff reconstruction of what changed across 1.0 → 1.1 → 1.2 → 1.3 → 1.4, including a **breaking change**: the `CXRServiceBridge()` no-argument constructor used throughout this doc set's examples was replaced by `CXRServiceBridge(Context)` starting in v1.1.

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
    implementation("com.rokid.cxr:cxr-service-bridge:1.4")
}
