---
name: orbit-mvi
description: "Implement, review, or improve Orbit MVI in Android and KMP apps. Use when working with ContainerHost, intent { }, reduce { }, and postSideEffect; wiring Orbit with Compose or SwiftUI; or writing unit tests with orbit-test. Covers state management, side effects, and multiplatform MVI patterns."
---
# Orbit MVI Skill

Orbit MVI is a lightweight, opinionated MVI framework for Kotlin Multiplatform.
It enforces a strict unidirectional data flow:

```
User Intent → ViewModel (intent block) → reduce / postSideEffect → State / SideEffect → UI
```

This skill covers the full development cycle: setup, ViewModel authoring,
Compose integration, testing, and multiplatform patterns.

---

## 1. Dependency Setup

Always confirm which artifact the user needs before adding dependencies.

### Android-only (Jetpack ViewModel)
```kotlin
// build.gradle.kts
dependencies {
    implementation("org.orbit-mvi:orbit-viewmodel:<VERSION>")   // ViewModel + ContainerHost
    implementation("org.orbit-mvi:orbit-compose:<VERSION>")     // collectAsState / collectSideEffect
    testImplementation("org.orbit-mvi:orbit-test:<VERSION>")    // runTest / test {}
}
```

### Kotlin Multiplatform (KMP)
```kotlin
// shared/build.gradle.kts
kotlin {
    sourceSets {
        commonMain.dependencies {
            implementation("org.orbit-mvi:orbit-core:<VERSION>")
        }
        androidMain.dependencies {
            implementation("org.orbit-mvi:orbit-viewmodel:<VERSION>")
        }
    }
    commonTest.dependencies {
        implementation("org.orbit-mvi:orbit-test:<VERSION>")
    }
}
```

> **Version**: Always check [GitHub releases](https://github.com/orbit-mvi/orbit-mvi/releases)
> for the latest stable version. As of mid-2024 the stable series is **7.x**.

---

## 2. Core Concepts

| Concept | Description |
|---|---|
| **State** | A single immutable data class representing the full UI state |
| **SideEffect** | One-shot events (navigation, toasts, dialogs) that must not live in State |
| **Intent** | A user action or system event; maps to an `intent {}` block in the ViewModel |
| **Container** | Orbit's internal state machine; created via `container(initialState)` |
| **ContainerHost** | Interface that exposes the `container`; implemented by the ViewModel |

### State vs SideEffect — decision rule
- **State**: anything the UI must *display* (loading flag, list items, error message text)
- **SideEffect**: anything the UI must *do once* (navigate, show snackbar, play haptic)

---

## 3. Defining State and SideEffect

```kotlin
// Always data class; always immutable; always has sensible defaults
data class LoginState(
    val email: String = "",
    val password: String = "",
    val isLoading: Boolean = false,
    val emailError: String? = null,
)

// Always sealed interface / sealed class
sealed interface LoginSideEffect {
    data object NavigateToHome : LoginSideEffect
    data class ShowError(val message: String) : LoginSideEffect
}
```

**Rules:**
- State must be a `data class` (enables structural equality for diffing).
- SideEffect must be `sealed interface` or `sealed class` (exhaustive `when`).
- Never put navigation destinations or one-shot UI events in State.

---

## 4. ViewModel (ContainerHost)

```kotlin
class LoginViewModel(
    private val authRepository: AuthRepository,
) : ViewModel(), ContainerHost<LoginState, LoginSideEffect> {

    override val container = container<LoginState, LoginSideEffect>(LoginState())

    // --- Intents ---

    fun onEmailChanged(value: String) = intent {
        reduce { state.copy(email = value, emailError = null) }
    }

    fun onPasswordChanged(value: String) = intent {
        reduce { state.copy(password = value) }
    }

    fun onLoginClicked() = intent {
        reduce { state.copy(isLoading = true) }

        val result = authRepository.login(state.email, state.password)

        reduce { state.copy(isLoading = false) }

        when (result) {
            is Result.Success -> postSideEffect(LoginSideEffect.NavigateToHome)
            is Result.Failure -> postSideEffect(
                LoginSideEffect.ShowError(result.message)
            )
        }
    }
}
```

### Key rules for the `intent {}` block
1. **`reduce {}`** — the *only* place where State is mutated. Always returns the new state.
   `state` inside `reduce` is the *current* guaranteed-latest snapshot.
2. **`postSideEffect()`** — emits a one-shot side effect downstream.
3. **`intent {}` is a coroutine scope** — suspend functions, `withContext`, `async`, etc. all work.
4. **Never** mutate external state or call `reduce` outside of `intent {}`.
5. **Parallelism**: by default, intents run sequentially per container. For parallel execution
   use `intent(registerIdling = false) {}` or pass a custom `CoroutineScope`.

---

## 5. Jetpack Compose Integration

```kotlin
@Composable
fun LoginScreen(viewModel: LoginViewModel = viewModel()) {

    // Collects StateFlow; recomposes only on actual state changes
    val state by viewModel.collectAsState()

    // Processes each SideEffect exactly once
    viewModel.collectSideEffect { sideEffect ->
        when (sideEffect) {
            LoginSideEffect.NavigateToHome  -> navController.navigate("home")
            is LoginSideEffect.ShowError    -> snackbarHostState.showSnackbar(sideEffect.message)
        }
    }

    LoginContent(
        state    = state,
        onEmail  = viewModel::onEmailChanged,
        onPassword = viewModel::onPasswordChanged,
        onLogin  = viewModel::onLoginClicked,
    )
}

// Pure, stateless composable — testable in isolation
@Composable
private fun LoginContent(
    state: LoginState,
    onEmail: (String) -> Unit,
    onPassword: (String) -> Unit,
    onLogin: () -> Unit,
) {
    // ...
}
```

**Always** separate the stateful `Screen` composable from the stateless `Content` composable.
The `Content` composable receives only plain data and lambdas — never the ViewModel directly.

---

## 6. Testing with orbit-test

Orbit's test library provides `runTest {}` + `test {}` that give deterministic,
coroutine-aware assertions without mocks for the container itself.

```kotlin
class LoginViewModelTest {

    @Test
    fun `email change updates state and clears error`() = runTest {
        val viewModel = LoginViewModel(fakeAuthRepository())

        viewModel.test {
            // Initial state assertion (optional)
            expectInitialState()

            // Trigger intent
            viewModel.onEmailChanged("test@example.com")

            // Assert exact state transition
            expectState { copy(email = "test@example.com", emailError = null) }
        }
    }

    @Test
    fun `login failure posts ShowError side effect`() = runTest {
        val repo = fakeAuthRepository(failWith = "Invalid credentials")
        val viewModel = LoginViewModel(repo)

        viewModel.test {
            expectInitialState()
            viewModel.onLoginClicked()

            expectState { copy(isLoading = true) }
            expectState { copy(isLoading = false) }
            expectSideEffect(LoginSideEffect.ShowError("Invalid credentials"))
        }
    }
}
```

### Test DSL cheat sheet

| Method | Purpose |
|---|---|
| `expectInitialState()` | Asserts the first emitted state equals `initialState` |
| `expectState { copy(...) }` | Asserts the next state matches the lambda result applied to the *previous* state |
| `expectState(LoginState(...))` | Asserts exact state value |
| `expectSideEffect(...)` | Asserts exact next side effect |
| `expectNoSideEffect()` | Asserts no side effect was emitted |
| `cancelAndIgnoreRemainingEvents()` | Tear down without failing on remaining events |

---

## 7. KMP / iOS Integration

For KMP targets, use `orbit-core` in `commonMain` and expose the container to SwiftUI
via a thin wrapper that converts `StateFlow` to `@Published`.

```kotlin
// commonMain — plain KMP class, no Android ViewModel
class LoginPresenter(
    private val authRepository: AuthRepository,
    coroutineScope: CoroutineScope,
) : ContainerHost<LoginState, LoginSideEffect> {

    override val container = container<LoginState, LoginSideEffect>(
        initialState = LoginState(),
        settings = Container.Settings(exceptionHandler = null),
        coroutineScope = coroutineScope,
    )

    fun onLoginClicked() = intent { /* ... */ }
}
```

```swift
// iOS — ObservableObject wrapper
@MainActor
class LoginObservable: ObservableObject {
    @Published var state: LoginState = LoginState()
    private let presenter: LoginPresenter
    private var cancellables: Set<AnyCancellable> = []

    init(presenter: LoginPresenter) {
        self.presenter = presenter
        // Observe Kotlin StateFlow via SKIE or KMP-NativeCoroutines
        presenter.container.stateFlow
            .toPublisher()
            .receive(on: RunLoop.main)
            .sink { [weak self] in self?.state = $0 }
            .store(in: &cancellables)
    }

    func onLoginClicked() { presenter.onLoginClicked() }
}
```

> Use **SKIE** (by Touchlab) or **KMP-NativeCoroutines** to bridge `StateFlow`/`SharedFlow`
> to Swift's `AsyncStream` or Combine publishers.

---

## 8. Common Patterns

### Loading + Error pattern
```kotlin
data class UiState(
    val items: List<Item> = emptyList(),
    val isLoading: Boolean = false,
    val error: String? = null,
)

fun loadItems() = intent {
    reduce { state.copy(isLoading = true, error = null) }
    runCatching { repository.getItems() }
        .onSuccess { items -> reduce { state.copy(items = items, isLoading = false) } }
        .onFailure { e -> reduce { state.copy(isLoading = false, error = e.message) } }
}
```

### Debounced search
```kotlin
private var searchJob: Job? = null

fun onQueryChanged(query: String) = intent {
    reduce { state.copy(query = query) }
    searchJob?.cancel()
    searchJob = viewModelScope.launch {
        delay(300)
        val results = repository.search(query)
        reduce { state.copy(results = results) }
    }
}
```

### Confirmation dialog (State-driven)
```kotlin
data class DeleteState(
    val showConfirmDialog: Boolean = false,
    val itemToDelete: Item? = null,
)

fun onDeleteRequested(item: Item) = intent {
    reduce { state.copy(showConfirmDialog = true, itemToDelete = item) }
}

fun onDeleteConfirmed() = intent {
    val item = state.itemToDelete ?: return@intent
    reduce { state.copy(showConfirmDialog = false, itemToDelete = null) }
    repository.delete(item)
}

fun onDeleteCancelled() = intent {
    reduce { state.copy(showConfirmDialog = false, itemToDelete = null) }
}
```

---

## 9. Anti-patterns to Avoid

| Anti-pattern | Why it's wrong | Correct approach |
|---|---|---|
| `reduce` called outside `intent {}` | Not thread-safe, breaks unidirectional flow | Always wrap in `intent {}` |
| Navigation route strings in State | State would carry stale nav commands on recomposition | Use `SideEffect` for navigation |
| Multiple ViewModels sharing a container | Containers are not designed for shared ownership | Use a shared repository / `SavedStateHandle` |
| Calling `intent {}` inside `reduce {}` | `reduce` is synchronous; no coroutine context | Move the `intent` call before or after `reduce` |
| Fat State (deeply nested objects) | Hard to diff, causes over-recomposition | Flatten State; use `sealed interface` for sub-states |
| Exposing `container` directly to UI | UI can mutate state bypassing intents | Only expose `intent` functions and `collectAsState()` |

---

## 10. Quick Reference

```
ContainerHost<State, SideEffect>
    └── container = container(initialState)

intent { }
    ├── state                          → read current state (snapshot)
    ├── reduce { state.copy(...) }     → emit new state
    └── postSideEffect(effect)         → emit side effect

// Compose
collectAsState()                       → State as Compose State<T>
collectSideEffect { }                  → one-shot side effect handler

// Testing
viewModel.test {
    expectInitialState()
    expectState { copy(...) }
    expectSideEffect(...)
}
```
