---
name: android-auto-fix-loop
description: Iteratively diagnoses and resolves Android build failures, Gradle dependency conflicts, and Kotlin compiler errors.
---

# Android Auto-Fix Loop Expert

## Instructions
Use a systematic, iterative loop to diagnose and fix deep build or runtime errors in Android.
1. **Log Analysis**: Read output from `./gradlew assembleDebug --stacktrace`, Logcat, or Kotlin compiler logs.
2. **Targeted Fixes**: Apply fixes incrementally. For Gradle, this often involves updating version catalogs (`libs.versions.toml`) or resolving namespace conflicts.
3. **Dependency Conflict Resolution**: Use `./gradlew :app:dependencies` to identify conflicts, especially when using KSP or the Compose compiler.
4. **Validation**: Re-run the build after every fix attempt to ensure the issue is resolved without introducing new ones.

## Example Prompts
- "I'm getting a 'Duplicate classes' error in Gradle. Run an auto-fix loop to resolve it."
- "My build is failing with a conflicting version of 'kotlinx-serialization'. Diagnose and fix the version catalog."
- "Build fails with 'Namespace not specified'. Iteratively debug and fix the build.gradle.kts files."

## Expected Output
A structured log of diagnostic steps taken, the specific fixes applied to Gradle configurations or version catalogs, and a final confirmation that the build succeeds.

## Edge Cases & Common Mistakes
- **Blindly Deleting Folders**: Running `./gradlew clean` without reading logs often masks the root cause. Diagnose first.
- **Forcing Dependency Versions**: Overriding dependencies without checking compatibility can break the app at runtime (e.g., mismatch between Room and SQLite).
- **Ignoring SDK Requirements**: Forgetting that new Android versions often require bumping `compileSdk` or `targetSdk`.

## Review Checklist
- [ ] Error logs fully parsed and root cause identified.
- [ ] Fixes applied incrementally and safely.
- [ ] `./gradlew clean` executed if necessary.
- [ ] Build succeeds on the CLI.
