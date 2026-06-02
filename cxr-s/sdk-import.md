# SDK Import

_Source: https://custom.rokid.com/prod/rokid_web/57e35cd3ae294d16b1b8fc8dcbb1b7c7/ (Chinese, fetched 2026-05-29). Version pin updated to match Maven `cxr-service-bridge` metadata (last updated 2026-05-22)._

This chapter uses Kotlin DSL (`build.gradle.kts`) as an example.

## Configure Maven Repository

The CXR-S SDK uses Maven for online SDK package management.

Maven repository address: `https://maven.rokid.com/repository/maven-public/`

Locate `settings.gradle.kts` and add the Maven repository to the `repositories` section inside the `dependencyResolutionManagement` node.

```kotlin
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
```

## Dependency Import

CXR-S SDK package: `com.rokid.cxr:cxr-service-bridge:1.0-20260212.103714-88`

> **Note:** This is a SNAPSHOT build. The `<release>` and `<latest>` tags in the Maven metadata point to `1.0-20260212.103714-88` (metadata `lastUpdated` 2026-05-22). There is no stable `1.0` release artifact — always use the latest SNAPSHOT coordinate above.

Add the dependency in the `dependencies` node of `build.gradle.kts`.

> **Note:** The SDK requires `minSdk` ≥ 28.

```kotlin
// ...Other Settings
android {
    // ...Other Settings
    defaultConfig {
        // ...Other Settings
        minSdk = 28
    }
    // ...Other Settings
}
dependencies {
    // ...Other Settings
    implementation("com.rokid.cxr:cxr-service-bridge:1.0-20260212.103714-88")
}
```
