---
name: android-migration-planner
description: Plan and execute high-stakes migrations, such as XML to Compose, RxJava to Coroutines, or Gradle Groovy to KTS.
---

# Android Migration Planner

## Instructions
Handle significant architectural shifts and version upgrades methodically to avoid regressions.
1. **Audit Phase**: Assess the current state using `./gradlew lint` and identify deprecated APIs or legacy patterns.
2. **Strategy**: Decide between a "strangler fig" approach (migrating feature by feature) or a wholesale rewrite for smaller modules.
3. **Execution**: Follow modern defaults. When migrating to Compose, use `AndroidViewBinding` or `ComposeView` for interoperability during the transition. When migrating to Coroutines, map `Observable` to `Flow`.
4. **Validation**: Ensure type safety and test coverage are maintained throughout the migration.

## Example Prompts
- "Plan a migration from XML Views to Jetpack Compose for the 'Settings' screen with deep linking requirements."
- "We are migrating from RxJava to Coroutines. Audit the 'auth' module and rewrite its reactive logic."
- "Upgrade this project to AGP 9.0 and Kotlin 2.2. Identify any deprecated Gradle DSL or compiler flags that need updating."

## Expected Output
A comprehensive migration document outlining the current vs. target state, followed by step-by-step code refactoring that maintains business logic while upgrading the architecture.

## Edge Cases & Common Mistakes
- **Big Bang Migrations**: Trying to migrate an entire app's UI in one PR, leading to merge conflicts and untraceable bugs.
- **Leaving Dead Code**: Not fully removing old XML layouts or RxJava dependencies after the migration is "complete."
- **Ignoring Binary Compatibility**: Forgetting that some migrations might break library consumers if the project is a library.

## Review Checklist
- [ ] Migration strategy explicitly defined (incremental vs. wholesale).
- [ ] Legacy APIs correctly replaced with modern equivalents (Compose, Flow).
- [ ] Target architecture (Hilt, Compose) correctly implemented.
- [ ] No regression in core functionality.
