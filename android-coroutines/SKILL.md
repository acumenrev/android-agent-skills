---
name: android-coroutines
description: "Expert guidance on implementing asynchronous logic, reactive streams, and lifecycle-safe concurrency in Android using Kotlin Coroutines and Flow."
---

# Android Coroutines Expert

## Instructions

Implement high-performance, lifecycle-safe asynchronous logic and reactive streams in Android using Kotlin Coroutines and Flow. Adhere to Kotlin 2.2+, AGP 9.0+, and modern structured concurrency principles.

### 1. Structured Concurrency & Scopes
*   **ViewModel Scoping**: Use `viewModelScope` for coroutines initiated within ViewModels.
*   **Lifecycle Safety**: In UI components (Activities/Fragments), always use `repeatOnLifecycle(Lifecycle.State.STARTED)` to collect flows. Avoid collecting directly in `lifecycleScope.launch`.
*   **GlobalScope Prohibition**: Never use `GlobalScope`. For tasks that must survive a specific screen, inject an `applicationScope` tied to the Application lifecycle.

### 2. Dispatchers & Main-Safety
*   **Dispatcher Injection**: Always inject `CoroutineDispatcher` via constructors (e.g., `ioDispatcher: CoroutineDispatcher = Dispatchers.IO`) to ensure testability.
*   **Main-Safe Repositories**: Every `suspend` function in the Data/Domain layer must be main-safe. Use `withContext(ioDispatcher)` internally to shift execution away from the main thread.
*   **Parallelism**: Use `coroutineScope` or `supervisorScope` with `async/await` for parallel operations. Use `supervisorScope` when child failures should not cancel siblings.

### 3. Reactive Streams with Flow
*   **State & Shared Flow**: Expose UI state as `StateFlow` (using `asStateFlow()`) and one-time events as `SharedFlow`. Never expose `Mutable` versions publicly.
*   **Callback Conversion**: Use `callbackFlow` to adapt listener-based APIs to Flow. Always use `awaitClose` to unregister listeners.
*   **Operator Selection**: Use `collectLatest` for UI updates where only the newest data matters. Use `flowOn` to specify the execution context for upstream operators.

### 4. Error Handling & Cancellability
*   **Cancellation Awareness**: Ensure long-running loops are cooperative by calling `ensureActive()` or `yield()`.
*   **Exception Handling**: Never catch `Exception` without rethrowing `CancellationException`. Use `CoroutineExceptionHandler` only for top-level coroutines in a scope.
*   **Supervision**: Use `SupervisorJob` or `supervisorScope` when you want to handle failures of individual child coroutines independently.

## Example Prompts

*   "Convert this callback-based Location API into a cold Flow using `callbackFlow`, ensuring proper cleanup and lifecycle awareness."
*   "Optimize this long-running data processing task in the Repository to be cooperatively cancellable and main-safe using injected dispatchers."
*   "Implement a robust error handling strategy for parallel network requests using `supervisorScope` and `async/await`, ensuring that one failing request doesn't cancel the others."

## Expected Output

*   Thread-safe and memory-efficient asynchronous code following structured concurrency rules.
*   Production-ready Flow implementations with proper error handling and lifecycle integration.
*   Testable code with support for `StandardTestDispatcher` and `runTest`.

## Edge Cases & Common Mistakes

*   **Swallowing Cancellation**: Catching `Exception` (or `Throwable`) and not rethrowing `CancellationException`, which prevents the coroutine from stopping when its scope is cancelled.
*   **Hardcoded Dispatchers**: Using `Dispatchers.IO` or `Dispatchers.Default` directly in logic, making it impossible to swap them for `TestDispatcher` during unit testing.
*   **Unsafe Flow Collection**: Collecting flows in the UI without `repeatOnLifecycle`, which can lead to "collection while in background" issues, wasting resources or causing crashes.
*   **Blocking the Main Thread**: Performing heavy I/O or computation inside a `suspend` function without using `withContext` to move to a background dispatcher.

## Review Checklist

- [ ] All `suspend` functions in repositories are main-safe.
- [ ] Dispatchers are injected, not hardcoded.
- [ ] UI flow collection uses `repeatOnLifecycle`.
- [ ] `MutableStateFlow` and `MutableSharedFlow` are encapsulated.
- [ ] Long-running tasks check for cancellation using `ensureActive()`.
- [ ] `CancellationException` is never swallowed.
- [ ] `callbackFlow` properly uses `awaitClose` for cleanup.
- [ ] `GlobalScope` is not used anywhere in the codebase.
