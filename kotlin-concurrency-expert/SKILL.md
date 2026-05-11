---
name: kotlin-concurrency-expert
description: "Kotlin Coroutines review and remediation for Android. Use when asked to review concurrency usage, fix coroutine-related bugs, improve thread safety, or resolve lifecycle issues."
---
# Kotlin Concurrency Expert

## Instructions

Review and implement Kotlin Coroutines in Android following structured concurrency and lifecycle safety principles.

### 1. Structured Concurrency & Scoping
Always use the appropriate scope to avoid memory leaks and zombie coroutines.
*   **ViewModel**: Use `viewModelScope` for operations that should live as long as the screen.
*   **UI (Activity/Fragment)**: Use `lifecycleScope` for UI-bound operations.
*   **Application**: Inject a custom `CoroutineScope` bound to the `Application` lifecycle for long-running background tasks (e.g., syncing).
*   **Avoid**: Never use `GlobalScope` or `NonCancellable` unless absolutely necessary and documented.

### 2. Main-Safe Suspend Functions
Suspend functions should be "main-safe," meaning they can be called from the main thread without blocking it.
*   **Pattern**: Use `withContext(Dispatchers.IO)` or `withContext(Dispatchers.Default)` inside the function to switch threads.
*   **Injection**: Inject `CoroutineDispatcher` via the constructor to allow for easier unit testing.

```kotlin
class UserRepository @Inject constructor(
    private val ioDispatcher: CoroutineDispatcher // Injected via Hilt
) {
    suspend fun fetchData() = withContext(ioDispatcher) {
        // Network or DB work
    }
}
```

### 3. Lifecycle-Aware Flow Collection
Use the `repeatOnLifecycle` or `collectAsStateWithLifecycle` APIs to ensure flows are only collected when the UI is in a specific state (usually `STARTED`).
*   **Benefit**: Automatically stops collection when the app is in the background, saving resources and preventing UI updates on a destroyed view.

### 4. Exception Handling
*   **CoroutineExceptionHandler**: Use for top-level exception handling in a scope.
*   **try/catch**: Use inside `launch` or `async` for granular error handling.
*   **CancellationException**: Never swallow `CancellationException`. Always rethrow it to allow the coroutine system to function correctly.

### 5. Shared State & Thread Safety
*   **StateFlow/SharedFlow**: Use these for reactive state management instead of manual listeners.
*   **Mutex**: Use `kotlinx.coroutines.sync.Mutex` instead of `synchronized` blocks for non-blocking mutual exclusion.

## Example Prompts

1.  "Review this ViewModel and ensure all coroutines are properly scoped and exceptions are handled correctly."
2.  "How do I convert this legacy callback-based API into a cold Flow using `callbackFlow`?"
3.  "My app is crashing with a `JobCancellationException` when I rotate the screen. Help me fix the lifecycle collection logic."

## Expected Output

The user should receive:
*   Refactored code using `viewModelScope` or `lifecycleScope`.
*   Implementation of main-safe suspend functions with injected dispatchers.
*   Flow collection logic using `repeatOnLifecycle` or `collectAsStateWithLifecycle`.
*   A clear strategy for handling exceptions across different coroutine scopes.
*   Thread-safe implementations for shared state using `Mutex` or `StateFlow`.

## Edge Cases & Common Mistakes

*   **Blocking the Main Thread**: Running heavy computations or I/O directly in `Dispatchers.Main` or without `withContext`.
*   **Leaking Subscriptions**: Collecting flows in `lifecycleScope.launch` without `repeatOnLifecycle`, which keeps the subscription active even when the view is not visible.
*   **Hardcoded Dispatchers**: Using `Dispatchers.IO` directly in classes, making them impossible to unit test with `TestDispatcher`.
*   **Swallowing Cancellation**: Catching `Exception` or `Throwable` and not rethrowing `CancellationException`.
*   **Async without Await**: Using `async` to start a coroutine but never calling `await()`, which can hide exceptions until the job is joined.
*   **launchWhenStarted**: Using the deprecated `launchWhenX` APIs, which pause execution but don't cancel the collection, potentially leading to wasted resources.

## Review Checklist

- [ ] Is `viewModelScope` or `lifecycleScope` used instead of custom/global scopes?
- [ ] Are all suspend functions main-safe using `withContext`?
- [ ] Are `CoroutineDispatcher`s injected via the constructor?
- [ ] Is `repeatOnLifecycle` or `collectAsStateWithLifecycle` used for UI flow collection?
- [ ] Are `CancellationException`s rethrown in catch blocks?
- [ ] Is `MutableStateFlow` properly encapsulated (private mutable, public read-only)?
- [ ] Is `callbackFlow` used correctly with `awaitClose` for callback APIs?
- [ ] Are heavy computations moved to `Dispatchers.Default`?
- [ ] Is `Mutex` used for thread-safe access to shared mutable state?
