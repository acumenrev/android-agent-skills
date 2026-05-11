---
name: compose-performance-audit
description: "Audit and improve Jetpack Compose runtime performance. Use when asked to diagnose slow rendering, janky scrolling, excessive recompositions, or performance issues in Compose UI."
---
# Compose Performance Expert

## Instructions

Audit Jetpack Compose performance end-to-end, from instrumentation to root-cause analysis and remediation.

### 1. Instrumentation & Profiling
Before optimizing, measure the performance on a **release build** with R8 enabled.
*   **Layout Inspector**: Use in Android Studio to observe recomposition counts and stability markers.
*   **Recomposition Highlights**: Enable in Compose tooling to visually identify active UI areas.
*   **Macrobenchmark**: Use for cold start, hot start, and scrolling performance metrics.

### 2. Identify Stability Issues
Compose skips recomposition if parameters are **stable**.
*   **Unstable Classes**: Classes with `var` properties or those containing unstable types like `List` or `Map` are considered unstable.
*   **Fix**: Use `@Immutable` or `@Stable` annotations for data classes, or use `kotlinx.collections.immutable` types.

### 3. Defer State Reads
Reading state during the **composition phase** triggers recomposition for the whole scope.
*   **Layout Phase**: Use lambda-based modifiers like `Modifier.offset { IntOffset(0, scrollState.value) }` to read state only during layout.
*   **Draw Phase**: Use `Modifier.drawBehind { ... }` for drawing operations.

### 4. Optimize Heavy Computations
*   **`remember`**: Cache expensive calculations (sorting, filtering) across recompositions.
*   **`derivedStateOf`**: Use when a state changes frequently (e.g., scroll position) but the UI only needs to react to a threshold.

### 5. Efficient Lazy Lists
*   **Stable Keys**: Always provide a unique `key` for `items` in `LazyColumn`/`LazyRow` to prevent unnecessary recompositions when the list changes.
*   **ContentType**: Use `contentType` to improve item reuse efficiency.

## Example Prompts

1.  "Audit this Composable for unnecessary recompositions and suggest stability improvements."
2.  "My screen is janking while scrolling through a long list of items. How can I optimize the LazyColumn performance?"
3.  "Why is this simple button triggering a recomposition of the entire screen when clicked? Help me debug the state reads."

## Expected Output

The user should receive:
*   A prioritized list of performance bottlenecks identified in the code.
*   Concrete code refactors demonstrating how to stabilize parameters or defer state reads.
*   Instructions on how to use profiling tools to verify the improvements.
*   A comparison of "Before" vs "After" patterns for common performance pitfalls.

## Edge Cases & Common Mistakes

*   **Profiling in Debug Mode**: Never trust performance metrics from a debug build. Debug builds have heavy instrumentation that masks real performance characteristics.
*   **Over-optimizing**: Don't wrap everything in `remember` or `derivedStateOf`. Only apply these when profiling shows a measurable cost or high recomposition counts.
*   **Unstable Lambda Captures**: Capturing unstable variables in a lambda passed to a child Composable makes the lambda itself unstable, causing the child to recompose. Use `remember` for the lambda or method references.
*   **Index as Key**: Using the list index as a `key` in `LazyColumn` is often worse than no key at all, as it causes every item to recompose when an item is inserted or removed at the beginning.
*   **Heavy Work in Composition**: Performing network calls, database queries, or heavy string formatting directly in a `@Composable` body is a critical mistake. These belong in a ViewModel.

## Review Checklist

- [ ] Is the performance being analyzed on a release build?
- [ ] Are all data classes passed to Composables marked as `@Immutable` or `@Stable`?
- [ ] Are `List`, `Map`, and `Set` parameters handled correctly (e.g., via `ImmutableList`)?
- [ ] Are frequent state reads deferred to layout/draw phases using lambdas?
- [ ] Do all `LazyColumn` items have stable, unique keys?
- [ ] Are expensive calculations wrapped in `remember`?
- [ ] Is `derivedStateOf` used where high-frequency state drives lower-frequency UI updates?
- [ ] Are modifiers allocated efficiently (avoiding recreation in high-frequency paths)?
