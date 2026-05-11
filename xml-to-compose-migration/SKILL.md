---
name: xml-to-compose-migration
description: "Expert guidance for converting Android XML layouts to Jetpack Compose. Covers layout mapping, state migration, incremental adoption strategies, and interoperability between View and Compose systems."
---

# XML to Compose Migration Expert

Expert guidance for systematically migrating Android XML-based layouts and View systems to **Jetpack Compose**. This skill covers layout translation, state hoisting, the removal of ViewBinding/DataBinding, and incremental adoption using `ComposeView` and `AndroidView`.

## Instructions

### 1. Layout Translation
- Map `LinearLayout` (vertical/horizontal) to `Column` or `Row`.
- Map `FrameLayout` to `Box`.
- Map `ConstraintLayout` (XML) to `ConstraintLayout` (Compose) or idiomatic nested Rows/Columns.
- Replace `RecyclerView` with `LazyColumn`, `LazyRow`, or `LazyVerticalGrid`.
- Replace `TextView`, `Button`, `ImageView` with their Compose equivalents (`Text`, `Button`, `Image`).

### 2. State Migration
- Replace `findViewById` and ViewBinding references with Compose `State` or `MutableState`.
- Convert `LiveData` or `Observable` subscriptions in Fragments/Activities to `collectAsStateWithLifecycle()` in Composable functions.
- Hoist state to the ViewModel and use lambdas for event callbacks (e.g., `onClicked: () -> Unit`).

### 3. Incremental Migration
- Use `ComposeView` to embed Compose content inside existing XML layouts.
- Use `AndroidView` to wrap legacy Views or third-party libraries (like Google Maps) that don't yet have a Compose equivalent.
- Follow a bottom-up approach: migrate individual components first, then full screens.

### 4. Theming & Resources
- Convert XML styles and themes to `MaterialTheme` (Material 3).
- Use `stringResource()`, `painterResource()`, and `colorResource()` to access existing Android resources from Compose.

## Example Prompts

### 1. Simple Layout Conversion
"Convert this `item_user.xml` layout to a Jetpack Compose Composable. It contains a `CardView` with a horizontal `LinearLayout` containing a circular `ImageView` and a vertical `LinearLayout` with two `TextView`s."

### 2. RecyclerView Migration
"Migrate this `RecyclerView.Adapter` and its XML item layout to a `LazyColumn`. The list should support item clicks and show a loading spinner if the list is empty."

### 3. Interoperability Setup
"How do I add a `ComposeView` to my existing `activity_main.xml` and set its content from my `MainActivity`? I want to pass a state from my ViewModel into the Composable."

## Expected Output
A comprehensive migration plan or code refactor that includes:
1. **Composable Functions**: Equivalent UI code following modern Compose best practices.
2. **State Management**: Updated ViewModel or State objects using `StateFlow` or `Compose State`.
3. **Resource Handling**: Proper use of `stringResource`, `dimensionResource`, etc.
4. **Interop Logic**: `ComposeView` or `AndroidView` setup if an incremental approach is requested.

## Edge Cases & Common Mistakes

- **Direct Translation of ConstraintLayout**: While `ConstraintLayout` exists in Compose, it's often more idiomatic and performant to use nested `Row`, `Column`, and `Box` with `Modifier.weight()` unless the constraints are truly complex.
- **State Hoisting Failure**: Migrating UI but keeping "View-like" state management (e.g., passing a `MutableState` into a child Composable instead of a value and a lambda) breaks unidirectional data flow.
- **RecyclerView vs LazyColumn**: Forgetting to use `items(items, key = { it.id })` in a `LazyColumn`. Keys are critical for performance and maintaining scroll state during list updates.
- **Lifecycle Awareness**: Collecting state in Compose using `collectAsState()` instead of `collectAsStateWithLifecycle()`. The latter is safer as it stops collection when the app is in the background.
- **Configuration Changes**: Forgetting that `remember { }` doesn't survive configuration changes. Use `rememberSaveable { }` or move the state to a `ViewModel`.

## Review Checklist

- [ ] Are all XML components mapped to their correct Compose equivalents?
- [ ] Is state properly hoisted to the ViewModel or a stateful parent?
- [ ] Are `Lazy` components used for lists and grids?
- [ ] Is `collectAsStateWithLifecycle()` used for state observation?
- [ ] Are resource lookups using Compose-specific functions (`stringResource`, etc.)?
- [ ] Is the migration following the project's Material 3 theme?
- [ ] Are click handlers passed as lambda parameters?
- [ ] Have `ViewBinding` or `DataBinding` references been removed from the migrated code?
- [ ] Are `Preview` functions included for UI verification?
- [ ] If interop is used, is `ComposeView` or `AndroidView` configured correctly with the lifecycle?
