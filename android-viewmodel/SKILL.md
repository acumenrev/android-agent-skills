---
name: android-viewmodel
description: "Best practices for implementing Android ViewModels, specifically focused on StateFlow for UI state and SharedFlow for one-off events."
---

# Android ViewModel & State Management Expert

Expert guidance on implementing **ViewModels** using **StateFlow**, **SharedFlow**, and **collectAsStateWithLifecycle**, following modern reactive architecture patterns.

## Instructions

Use `ViewModel` to hold UI state and encapsulate business logic. It must outlive configuration changes and remain independent of the UI framework.

### 1. UI State (StateFlow)
Represent the persistent state of the UI (e.g., `Loading`, `Success(data)`, `Error`).

*   **Type**: `StateFlow<UiState>`.
*   **Exposure**: Expose as a read-only `StateFlow` backing a private `MutableStateFlow`.
*   **Updates**: Use `.update { oldState -> ... }` for atomic updates.

```kotlin
data class MyUiState(val items: List<String> = emptyList(), val isLoading: Boolean = false)

class MyViewModel : ViewModel() {
    private val _uiState = MutableStateFlow(MyUiState(isLoading = true))
    val uiState: StateFlow<MyUiState> = _uiState.asStateFlow()
}
```

### 2. One-Off Events (SharedFlow)
Use for transient events like "Show Snackbar", "Navigate", or "Show Toast".

*   **Type**: `SharedFlow<UiEvent>`.
*   **Configuration**: Must use `replay = 0` to prevent events from re-triggering on rotation.

```kotlin
sealed interface MyUiEvent {
    data class ShowError(val message: String) : MyUiEvent
}

private val _uiEvent = MutableSharedFlow<MyUiEvent>(replay = 0)
val uiEvent: SharedFlow<MyUiEvent> = _uiEvent.asSharedFlow()
```

### 3. Collecting in Compose
Always use `collectAsStateWithLifecycle()` to ensure the collector is lifecycle-aware and avoids unnecessary resource consumption when the app is in the background.

```kotlin
@Composable
fun MyScreen(viewModel: MyViewModel = hiltViewModel()) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()
    
    LaunchedEffect(Unit) {
        viewModel.uiEvent.collect { event ->
            // Handle one-off events
        }
    }
}
```

### 4. ViewModelScope & Operations
Start all coroutines in `viewModelScope`. Delegate data fetching to Repositories and domain logic to UseCases.

```kotlin
fun loadData() {
    viewModelScope.launch {
        _uiState.update { it.copy(isLoading = true) }
        val result = repository.getData()
        _uiState.update { it.copy(items = result, isLoading = false) }
    }
}
```

## Example Prompts

- "Help me design a ViewModel that handles a paginated list of items with pull-to-refresh functionality using StateFlow."
- "Show me the best way to handle navigation events from a ViewModel to a Compose screen using SharedFlow."
- "How do I use SavedStateHandle in my ViewModel to persist simple UI state (like search queries) across process death?"

## Expected Output

You should receive a robust `ViewModel` implementation with private mutable and public immutable state flows, an event flow for one-offs, and clear `viewModelScope` usage for launching business logic.

## Edge Cases & Common Mistakes

- **State Re-triggering**: Using `StateFlow` for navigation or snackbars will cause them to re-trigger on screen rotation. Use `SharedFlow` with `replay = 0`.
- **MutableStateFlow.value = ...**: Avoid direct assignment of `.value`. Use `.update { ... }` to avoid race conditions when multiple coroutines update the same state.
- **Collecting in LaunchedEffect**: For `StateFlow`, use `collectAsStateWithLifecycle`. For `SharedFlow` (events), use `LaunchedEffect` but ensure you collect on the correct lifecycle state if needed.
- **Long-running work in Init**: Avoid performing heavy initialization or network calls directly in the `init` block. Trigger them explicitly or use `stateIn` with `SharingStarted.WhileSubscribed`.

## Review Checklist

- [ ] Does the ViewModel expose read-only `StateFlow` and `SharedFlow`?
- [ ] Is `collectAsStateWithLifecycle()` used in the Compose layer?
- [ ] Are all coroutines launched in `viewModelScope`?
- [ ] Does the UI state use a `data class` for easy `copy()` updates?
- [ ] Is `SavedStateHandle` used for surviving process death if required?
