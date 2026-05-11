---
name: rxjava-to-coroutines-migration
description: "Expert guidance for migrating asynchronous code from RxJava to Kotlin Coroutines and Flow. Provides mapping for types, operators, and schedulers to ensure a safe and idiomatic transition."
---

# RxJava to Coroutines Migration Expert

Expert guidance for refactoring Android or Kotlin codebases from RxJava (Observables, Singles, Completables) to **Kotlin Coroutines and Flow**. This skill ensures a safe, systematic, and idiomatic transition while preserving business logic and error handling.

## Instructions

### 1. Base Type Mapping
- **`Single<T>`** -> `suspend fun ...(): T` (One-shot operation).
- **`Maybe<T>`** -> `suspend fun ...(): T?` (Optional one-shot).
- **`Completable`** -> `suspend fun ...()` (Operation with no return value).
- **`Observable<T>` / `Flowable<T>`** -> `Flow<T>` (Cold stream).

### 2. Schedulers to Dispatchers
- **`Schedulers.io()`** -> `Dispatchers.IO`.
- **`Schedulers.computation()`** -> `Dispatchers.Default`.
- **`AndroidSchedulers.mainThread()`** -> `Dispatchers.Main`.
- Replace `subscribeOn` and `observeOn` with `withContext` (for suspend functions) or `flowOn` (for Flows).

### 3. Subject to Hot Flow Mapping
- **`PublishSubject<T>`** -> `MutableSharedFlow<T>`.
- **`BehaviorSubject<T>`** -> `MutableStateFlow<T>` (requires initial value).
- **`ReplaySubject<T>`** -> `MutableSharedFlow<T>(replay = n)`.

### 4. Operator Translation
- `map`, `filter`, `zip`, `combineLatest` -> direct equivalents in Flow (`map`, `filter`, `zip`, `combine`).
- `flatMap` -> `flatMapMerge` (parallel) or `flatMapConcat` (sequential).
- `switchMap` -> `flatMapLatest`.
- `doOnNext` / `doOnSuccess` -> `onEach`.
- `onErrorReturn` / `onErrorResumeNext` -> `catch { emit(...) }`.

### 5. Execution & Lifecycle
- Replace `.subscribe()` with `launch { ... }` or `collect { ... }`.
- Ensure all coroutines are launched in an appropriate scope (e.g., `viewModelScope`).
- Use `repeatOnLifecycle` or `flowWithLifecycle` in UI layers (Fragments/Compose) for safety.

## Example Prompts

### 1. Repository Migration
"Migrate this `UserRepository` from RxJava to Coroutines. It currently returns `Single<User>` for fetching a profile and `Observable<List<Message>>` for a real-time message stream. Ensure it uses `Dispatchers.IO` internally."

### 2. ViewModel Refactoring
"Refactor this ViewModel to use `StateFlow` and `viewModelScope`. It currently uses a `CompositeDisposable` to manage several RxJava subscriptions that update `MutableLiveData` objects."

### 3. Complex Chain Conversion
"Convert this complex RxJava chain to a Kotlin Flow: `observable.flatMap { ... }.filter { ... }.debounce(300).distinctUntilChanged().observeOn(Main).subscribe(...)`."

## Expected Output
A systematic migration report or code refactor that includes:
1. **Updated Function Signatures**: `suspend` functions and `Flow<T>` return types.
2. **Idiomatic Operator Mapping**: Translation of RxJava operators to Flow counterparts.
3. **Structured Concurrency**: Proper use of `CoroutineScope` and cancellation logic.
4. **Error Handling**: Conversion of `onError` handlers to `try-catch` blocks or Flow `catch` operators.

## Edge Cases & Common Mistakes

- **Memory Leaks (Disposables)**: Forgetting to cancel a `Job` or use `viewModelScope` is the equivalent of forgetting to add a subscription to a `CompositeDisposable`. Always verify the scope lifecycle.
- **Blocking the Main Thread**: RxJava's `subscribeOn(Schedulers.io())` is often placed at the end of a chain. In Coroutines, the *caller* usually decides the dispatcher, but repositories should be "main-safe" by using `withContext(Dispatchers.IO)`.
- **Backpressure**: While RxJava has `Flowable`, Kotlin `Flow` handles backpressure natively through suspension. If an upstream produces faster than a downstream consumes, the upstream will simply suspend until the downstream is ready.
- **GlobalScope Usage**: Avoid using `GlobalScope` as it bypasses structured concurrency. Always use a supervisor scope or a lifecycle-aware scope.
- **StateFlow Initialization**: `BehaviorSubject` can be initialized without a value (though rare), but `StateFlow` *must* have an initial value. If no initial value is available, consider `SharedFlow` with `replay = 1` or a nullable StateFlow.

## Review Checklist

- [ ] Are `Single` and `Completable` converted to `suspend` functions?
- [ ] Are `Observable` and `Flowable` converted to `Flow`?
- [ ] Is `viewModelScope` used for launching coroutines in ViewModels?
- [ ] Is `withContext(Dispatchers.IO)` used for blocking I/O operations?
- [ ] Are RxJava operators correctly mapped to Flow equivalents?
- [ ] Is error handling performed via `try-catch` or `.catch {}`?
- [ ] Are `MutableStateFlow` or `MutableSharedFlow` used instead of Subjects?
- [ ] Is the UI layer using `collectAsStateWithLifecycle` (Compose) or `repeatOnLifecycle` (Views)?
- [ ] Have all `Disposables` and RxJava imports been removed?
