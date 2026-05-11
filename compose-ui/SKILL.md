---
name: compose-ui
description: "Best practices for building UI with Jetpack Compose. Focuses on state hoisting, detailed performance optimizations, and theming. Use when writing or refactoring Composable functions."
---
# Compose UI Expert

## Instructions

Build performant, reusable, and accessible UI components using Jetpack Compose. Follow these core principles and patterns.

### 1. State Hoisting & Unidirectional Data Flow (UDF)
Decouple your UI from its state logic to improve testability and reusability.
*   **Pattern**: Pass state down and events up.
*   **Stateless Composables**: Aim to make most Composables stateless by hoisting state to the caller or a ViewModel.
*   **ViewModel**: Use `collectAsStateWithLifecycle()` to observe state flows in a lifecycle-aware manner.

```kotlin
@Composable
fun UserProfileScreen(viewModel: UserViewModel = hiltViewModel()) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()
    
    UserProfileContent(
        user = uiState.user,
        onUpdateName = viewModel::updateName
    )
}
```

### 2. Standard Modifier Usage
Every public Composable should accept a `modifier` parameter to allow callers to adjust layout and behavior.
*   **Default Parameter**: Always provide `modifier: Modifier = Modifier` as the first optional parameter.
*   **Application**: Apply this `modifier` to the root layout element of the Composable.
*   **Concatenation**: Chains should be readable: `Modifier.fillMaxSize().padding(16.dp).clickable { }`.

### 3. Material Design 3 & Theming
Strictly adhere to the design system to ensure visual consistency and support for Dark Mode/Dynamic Color.
*   **Theming**: Use `MaterialTheme.colorScheme`, `MaterialTheme.typography`, and `MaterialTheme.shapes`.
*   **Hardcoded Values**: Avoid hardcoded colors (e.g., `Color.White`) or DP values that aren't tied to a consistent spacing grid.

### 4. Accessibility (a11y)
Ensure your UI is usable by everyone.
*   **Content Descriptions**: Provide meaningful descriptions for images and icons. Use `null` for purely decorative elements.
*   **Click Labels**: Use `Modifier.clickable(onClickLabel = "...")` to provide context for screen readers.
*   **Merge Descendants**: Use `Modifier.semantics(mergeDescendants = true)` for complex components that should be treated as a single focusable item.

### 5. Previews & Tooling
*   **Multi-Preview**: Use annotations to preview Light/Dark modes, different font scales, and screen sizes.
*   **Composition Local**: Use `CompositionLocal` sparingly (e.g., for themes or user settings) to avoid making the data flow implicit and hard to trace.

## Example Prompts

1.  "Create a stateless 'ProductCard' component using Material 3 that displays an image, title, price, and a favorite toggle button."
2.  "Refactor this Composable to hoist the state and make it more testable, showing both the stateless and stateful versions."
3.  "How do I implement an accessible custom top app bar that handles window insets and supports both light and dark modes?"

## Expected Output

The user should receive:
*   Stateless and stateful Composable implementations.
*   Idiomatic use of `Modifier` for layout and interaction.
*   Correct integration with `MaterialTheme`.
*   Accessibility enhancements (content descriptions, semantics).
*   Preview functions demonstrating the component in various states.

## Edge Cases & Common Mistakes

*   **State Leaks**: Holding state in a Composable that should be managed by a ViewModel or a higher-level caller.
*   **Hardcoded String Resources**: Always use `stringResource(id = R.string.label)` instead of hardcoded strings to support localization.
*   **Ignoring Window Insets**: Forgetting to handle status bar and navigation bar padding, leading to UI elements overlapping system bars. Use `Modifier.safeDrawingPadding()`.
*   **Over-nesting**: Using too many deep hierarchies (e.g., Row inside Column inside Box inside Row). Flatten layouts using `ConstraintLayout` or smarter use of `Modifier.weight()`.
*   **Missing Reusable Components**: Re-implementing standard patterns (like a primary button) multiple times instead of creating a single reusable Design System component.

## Review Checklist

- [ ] Does the Composable accept a `modifier` parameter and apply it to the root?
- [ ] Is state hoisted appropriately (Stateless vs Stateful)?
- [ ] Does it use `MaterialTheme` for colors, typography, and shapes?
- [ ] Are all images provided with `contentDescription` or explicitly marked decorative?
- [ ] Does it handle `WindowInsets` correctly (Edge-to-Edge)?
- [ ] Are strings retrieved from `res/values/strings.xml`?
- [ ] Is there a `@Preview` function for the Composable?
- [ ] Are frequent state reads optimized using lambda-based modifiers?
