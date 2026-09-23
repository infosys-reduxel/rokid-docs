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

> **Version note (2026-09-23):** the snapshot pinned above (`1.0-20260522.063600-105`) is the version this chapter was originally translated from. Rokid's public Maven repository now lists `cxr-service-bridge` release `1.4` (versions `1.0` → `1.1` → `1.2` → `1.3` → `1.4` were published between the original translation and 2026-09-22, per `maven-metadata.xml`). No official changelog for `cxr-service-bridge` 1.1–1.4 has been located — the SDK-selection landing page at `developerdoc.rokid.com/sdk` does not carry a dedicated CXR-S card, and `custom.rokid.com` (the prior detail-doc host) is offline. Pin to the latest release for new projects (`implementation("com.rokid.cxr:cxr-service-bridge:1.4")`) and verify against your own integration testing; the import steps and `minSdk` requirement below are otherwise unchanged.

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
