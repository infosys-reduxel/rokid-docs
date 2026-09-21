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

> **Note (2026-09-21):** Rokid's public Maven (`https://maven.rokid.com/repository/maven-public/com/rokid/cxr/cxr-service-bridge/maven-metadata.xml`) now lists `<release>1.3</release>` (`lastUpdated` 2026-09-18), with `1.1` and `1.2` also published in between — three releases ahead of the `1.0` example above. Rokid's own developer-portal example was still showing `1.0` as of this check, so the exact dependency coordinate for `1.3` is unconfirmed. TODO: confirm and update the import snippet once Rokid publishes updated guidance for the newer release.

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
