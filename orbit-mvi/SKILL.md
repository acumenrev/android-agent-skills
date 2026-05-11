---
name: orbit-mvi
description: "Expert guidance for implementing, reviewing, and testing Orbit MVI in Android and Kotlin Multiplatform (KMP) applications. Covers ContainerHost, intent/reduce/postSideEffect patterns, Compose integration, and unit testing with orbit-test."
---

# Orbit MVI Expert

Expert guidance for architecting and implementing robust, unidirectional data flow (UDF) using **Orbit MVI** in Android and Kotlin Multiplatform (KMP). Orbit is a lightweight, opinionated MVI framework that enforces strict separation of concerns through State, SideEffects, and Intents.

## Instructions

### 1. Core Architecture
- **State**: Use a single immutable `data class` to represent the full UI state. Provide sensible defaults.
- **SideEffect**: Use a `sealed interface` for one-shot events (navigation, snackbars, dialogs) that shouldn't persist in state.
- **ViewModel (ContainerHost)**: ViewModels must implement `ContainerHost<State, SideEffect>` and initialize a `container`.

### 2. Intent & State Mutation
- **intent { }**: All business logic and state transitions must occur within an `intent` block.
- **reduce { }**: The *only* way to update state. It provides a thread-safe snapshot of the current state.
- **postSideEffect(effect)**: Emits a one-shot event to the UI layer.

### 3. Jetpack Compose Integration
- Use `collectAsState()` to observe the state flow as Compose State.
- Use `collectSideEffect { }` to handle one-shot events exactly once.
- **Stateless Content**: Always separate the stateful `Screen` composable from a stateless `Content` composable that accepts plain data and lambdas.

### 4. Strong Modern Defaults (Android)
- **Hilt**: Inject repositories and use `@HiltViewModel`.
- **Kotlin 2.2+**: Leverage modern language features.
- **Coroutines**: Use `viewModelScope` and handle cancellation via `intent` blocks.

## Example Prompts

### 1. New Feature Implementation
"Help me implement a Search Screen using Orbit MVI. I need to handle a text query, a loading state, a list of results from a repository, and a side effect for showing an error toast if the search fails."

### 2. Testing Logic
"Write a unit test using `orbit-test` for my `LoginViewModel`. I want to verify that when `onLoginClicked` is called, the state first transitions to `isLoading = true`, then to `isLoading = false`, and finally posts a `NavigateToHome` side effect on success."

### 3. Refactoring to MVI
"Refactor this traditional MVVM ViewModel that uses multiple `MutableStateFlow`s for loading, error, and data into a single-container Orbit MVI implementation."

## Expected Output
A complete, idiomatic Orbit MVI implementation including:
1. **State and SideEffect** definitions (sealed interfaces/data classes).
2. **ViewModel** implementing `ContainerHost` with clear `intent` blocks.
3. **Jetpack Compose** UI code using `collectAsState` and `collectSideEffect`.
4. **Unit Tests** using `orbit-test` DSL (`expectInitialState`, `expectState`, `expectSideEffect`).

## Edge Cases & Common Mistakes

- **State vs. SideEffect Confusion**: Avoid putting "Show Toast" or "Navigate" flags in the `State`. This leads to "zombie" states where the event triggers again on recomposition or configuration change. Use `SideEffect` for one-shot actions.
- **Non-Thread-Safe Access**: Never read `container.stateFlow.value` directly inside an `intent` block for mutation. Always use the `state` property provided by the `reduce` lambda to ensure you are working with the latest snapshot.
- **Heavily Nested State**: Deeply nested `data class` state makes `copy()` calls verbose and error-prone. Keep state flat or use `sealed interface` to represent mutually exclusive sub-states (e.g., `Loading`, `Success`, `Error`).
- **Parallel Intents**: By default, `intent` blocks in a container run sequentially. If you need a long-running background task (like a socket listener) that shouldn't block other UI intents, launch it in a separate `CoroutineScope` or use `intent(registerIdling = false)`.

## Review Checklist

- [ ] Does the ViewModel implement `ContainerHost`?
- [ ] Is the state a single immutable `data class`?
- [ ] Are one-shot events handled via `SideEffect` and `postSideEffect`?
- [ ] Are all state changes performed inside `reduce { }` blocks?
- [ ] Is the Compose UI separating state collection from stateless rendering?
- [ ] Are unit tests using the `orbit-test` library for deterministic verification?
- [ ] Are sensible defaults provided for the initial state?
- [ ] Are Hilt and `viewModelScope` used appropriately?
