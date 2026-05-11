---
name: android-gradle-logic
description: "Expert guidance on setting up scalable Gradle build logic using Convention Plugins and Version Catalogs for AGP 9.0+."
---

# Android Gradle Build Logic Expert

This skill helps you configure a scalable, maintainable build system for Android apps using **Gradle Convention Plugins** and **Version Catalogs**, following the "Now in Android" (NiA) architecture patterns optimized for AGP 9.0+.

## Instructions

Stop copy-pasting code between `build.gradle.kts` files. Centralize build logic (Compose setup, Kotlin options, Hilt, etc.) in reusable plugins.

### 1. Project Structure
Ensure your project has a `build-logic` directory included in `settings.gradle.kts` as a composite build.

```text
root/
├── build-logic/
│   ├── convention/
│   │   ├── src/main/kotlin/
│   │   │   └── AndroidApplicationConventionPlugin.kt
│   │   └── build.gradle.kts
│   ├── build.gradle.kts
│   └── settings.gradle.kts
├── gradle/
│   └── libs.versions.toml
├── app/
│   └── build.gradle.kts
└── settings.gradle.kts
```

### 2. Configure `settings.gradle.kts`
Include the `build-logic` as a plugin management source.

```kotlin
// settings.gradle.kts
pluginManagement {
    includeBuild("build-logic")
    repositories {
        google()
        mavenCentral()
        gradlePluginPortal()
    }
}
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
    }
}
```

### 3. Define Dependencies in `libs.versions.toml`
Use the Version Catalog for both libraries *and* plugins. Note that with **AGP 9.0+**, Kotlin is built-in and versioned by AGP, though you can still specify versions for other Kotlin components.

```toml
[versions]
androidGradlePlugin = "9.0.0"
kotlin = "2.2.10"

[libraries]
# ...

[plugins]
android-application = { id = "com.android.application", version.ref = "androidGradlePlugin" }
android-library = { id = "com.android.library", version.ref = "androidGradlePlugin" }
# Custom convention plugins
myproject-android-application = { id = "myproject.android.application", version = "unspecified" }
myproject-android-compose = { id = "myproject.android.compose", version = "unspecified" }
```

### 4. Create a Convention Plugin
Inside `build-logic/convention/src/main/kotlin/AndroidApplicationConventionPlugin.kt`:

```kotlin
import com.android.build.api.dsl.ApplicationExtension
import org.gradle.api.Plugin
import org.gradle.api.Project
import org.gradle.kotlin.dsl.configure

class AndroidApplicationConventionPlugin : Plugin<Project> {
    override fun apply(target: Project) {
        with(target) {
            with(pluginManager) {
                apply("com.android.application")
            }

            extensions.configure<ApplicationExtension> {
                compileSdk = 35
                defaultConfig {
                    targetSdk = 35
                    minSdk = 26
                }
                compileOptions {
                    sourceCompatibility = JavaVersion.VERSION_21
                    targetCompatibility = JavaVersion.VERSION_21
                }
            }
        }
    }
}
```

Register it in `build-logic/convention/build.gradle.kts`:

```kotlin
gradlePlugin {
    plugins {
        register("androidApplication") {
            id = "myproject.android.application"
            implementationClass = "AndroidApplicationConventionPlugin"
        }
    }
}
```

### 5. Compose Convention Plugin (AGP 9.0+)
AGP 9.0 simplifies Compose setup by making the Compose compiler a built-in feature.

```kotlin
class AndroidComposeConventionPlugin : Plugin<Project> {
    override fun apply(target: Project) {
        with(target) {
            extensions.configure<ApplicationExtension> {
                buildFeatures {
                    compose = true
                }
                // Kotlin 2.2 / AGP 9.0 built-in compiler options
                composeOptions {
                    // compilerVersion is no longer required with built-in compiler
                }
            }
        }
    }
}
```

## Example Prompts

- "Help me set up a Gradle convention plugin for a multi-module Android project to share Hilt and Compose configuration."
- "Migrate my existing Groovy build files to Kotlin DSL with Version Catalogs and AGP 9.0."
- "How do I create a reusable 'android-library' convention plugin that enforces specific lint rules and minSdk versions across 20 modules?"

## Expected Output

You should receive a structured set of Gradle configuration files, including `libs.versions.toml`, the `build-logic` project structure, and the specific `ConventionPlugin` classes required to centralize your build logic.

## Edge Cases & Common Mistakes

- **Circular Dependencies in build-logic**: Avoid having convention plugins depend on each other in a way that creates a cycle. Use fine-grained plugins and apply them as needed.
- **Incorrect includeBuild Path**: Ensure the path in `settings.gradle.kts` (`includeBuild("build-logic")`) matches the actual folder name relative to the root.
- **Namespace Collisions**: Use unique package names for your convention plugin classes and unique IDs for the plugins themselves (e.g., `myproject.android.library`).
- **Ignoring Version Catalog in build-logic**: The `build-logic` project itself can use the version catalog. Ensure you have a `libs.versions.toml` accessible or symlinked if necessary, or use the standard `gradle/libs.versions.toml`.

## Review Checklist

- [ ] Does `settings.gradle.kts` use `includeBuild("build-logic")`?
- [ ] Are all common versions (AGP, Kotlin, etc.) defined in `libs.versions.toml`?
- [ ] Do custom plugins follow the naming convention `id("yourproject.android.*")`?
- [ ] Is `JavaVersion.VERSION_21` used for Kotlin 2.2 / AGP 9.0?
- [ ] Are module-level `build.gradle.kts` files reduced to just applying convention plugins and module-specific dependencies?
