---
name: compose-navigation
description: "Implement type-safe navigation in Jetpack Compose using Navigation Compose. Use when asked to set up navigation, pass arguments between screens, handle deep links, or structure multi-screen apps."
---
# Compose Navigation Expert

## Instructions

Implement type-safe navigation in Jetpack Compose applications using the Navigation Compose library. Follow these guidelines for modern, maintainable navigation.

### 1. Setup & Dependencies
Ensure you have the necessary dependencies for type-safe navigation and Kotlin serialization.

```kotlin
// build.gradle.kts
plugins {
    kotlin("plugin.serialization") version "2.1.0" // Latest for Kotlin 2.1+
}

dependencies {
    implementation("androidx.navigation:navigation-compose:2.8.5")
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.7.3")
}
```

### 2. Type-Safe Route Definitions
Use `@Serializable` data classes and objects to define routes. This replaces the legacy string-based route pattern.

```kotlin
import kotlinx.serialization.Serializable

@Serializable
object Home

@Serializable
data class ProductDetail(val productId: String)

@Serializable
object Settings
```

### 3. NavHost Implementation
Structure your `NavHost` using the type-safe `composable<T>` and `navigation<T>` extensions.

```kotlin
@Composable
fun AppNavHost(
    navController: NavHostController,
    modifier: Modifier = Modifier
) {
    NavHost(
        navController = navController,
        startDestination = Home,
        modifier = modifier
    ) {
        composable<Home> {
            HomeScreen(onProductClick = { id -> 
                navController.navigate(ProductDetail(id)) 
            })
        }
        composable<ProductDetail> { backStackEntry ->
            val route: ProductDetail = backStackEntry.toRoute()
            ProductDetailScreen(productId = route.productId)
        }
    }
}
```

### 4. ViewModel Integration
Retrieve navigation arguments directly in the ViewModel using `SavedStateHandle.toRoute<T>()`.

```kotlin
@HiltViewModel
class ProductViewModel @Inject constructor(
    savedStateHandle: SavedStateHandle
) : ViewModel() {
    private val route = savedStateHandle.toRoute<ProductDetail>()
    val productId = route.productId
}
```

### 5. Advanced Patterns
*   **Deep Links**: Define deep links directly on the `composable` call using `navDeepLink<T>`.
*   **Nested Graphs**: Group related screens using `navigation<GraphType>`.
*   **Navigation Options**: Use `popUpTo`, `launchSingleTop`, and `restoreState` for bottom navigation to maintain backstack integrity.

## Example Prompts

1.  "Set up a type-safe navigation graph for an app with a Home screen, a Search screen, and a User Profile screen that takes a 'userId' argument."
2.  "Show me how to implement a bottom navigation bar using Navigation Compose that preserves the state of each tab."
3.  "How do I handle a deep link that navigates directly to a specific order details screen using the new type-safe navigation APIs?"

## Expected Output

The user should receive:
*   A complete `NavHost` implementation using type-safe routes.
*   `@Serializable` route definitions for all screens.
*   Code demonstrating how to navigate and pass arguments.
*   ViewModel examples showing how to retrieve those arguments.
*   Clear instructions on necessary `build.gradle.kts` configuration.

## Edge Cases & Common Mistakes

*   **Passing Complex Objects**: Never pass large or complex objects (e.g., a `User` entity) in routes. Pass the unique ID and fetch the data in the destination's ViewModel. This keeps the URL/backstack small and ensures data consistency.
*   **Forgetting Serialization Plugin**: The Kotlin serialization plugin is mandatory for type-safe routes. Ensure it is applied in the `plugins` block.
*   **Manual String Parsing**: Avoid manually parsing navigation arguments from strings or bundle keys. Always use `.toRoute<T>()` for both `NavBackStackEntry` and `SavedStateHandle`.
*   **Incorrect Deep Link Setup**: Ensure the `basePath` in `navDeepLink` matches the intent-filter in `AndroidManifest.xml`.
*   **Navigating in Composition**: Do not call `navController.navigate()` directly in a Composable's body. Use `LaunchedEffect` or a callback to ensure it happens in response to an event.

## Review Checklist

- [ ] Are all routes defined as `@Serializable` data classes or objects?
- [ ] Is the `NavHost` using the type-safe `composable<T>` calls?
- [ ] Are arguments retrieved using `toRoute<T>()` in ViewModels or Composables?
- [ ] Is `modifier` passed and used in the root of the `NavHost`?
- [ ] For bottom nav, are `saveState`, `restoreState`, and `launchSingleTop` used correctly?
- [ ] Are complex objects avoided as navigation arguments?
- [ ] Are dependencies and plugins correctly configured in Gradle?
