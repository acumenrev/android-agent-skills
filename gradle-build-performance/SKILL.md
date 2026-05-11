---
name: gradle-build-performance
description: "Debug and optimize Android/Gradle build performance. Use when builds are slow, investigating CI/CD performance, or identifying compilation bottlenecks."
---
# Gradle Performance Expert

## Instructions

Optimize Android build performance by addressing initialization, configuration, and execution bottlenecks. Use modern AGP 9.0+ and Gradle features.

### 1. Enable Configuration Cache
The configuration cache significantly speeds up builds by caching the output of the configuration phase.
*   **Action**: Add `org.gradle.configuration-cache=true` to `gradle.properties`.
*   **Verification**: Ensure custom plugins and tasks are compatible by resolving any "Configuration cache" warnings.

### 2. Built-in Kotlin (AGP 9.0+)
AGP 9.0 introduces built-in Kotlin support, which removes the overhead of the Kotlin Gradle Plugin (KGP) for many projects.
*   **Action**: Ensure `android.builtInKotlin=true` is enabled (default in AGP 9.0).
*   **Benefit**: Faster project syncs and reduced configuration time.

### 3. Build Cache & Parallel Execution
*   **Build Cache**: Reuses task outputs from previous builds. Enable via `org.gradle.caching=true`.
*   **Parallel Execution**: Allows Gradle to run tasks in different modules simultaneously. Enable via `org.gradle.parallel=true`.

### 4. Dependency Optimization
*   **Avoid Dynamic Versions**: Never use `+` or `1.0.+` in dependency versions as it forces network checks on every build.
*   **KSP over kapt**: Kotlin Symbol Processing (KSP) is significantly faster than kapt. Migrate Dagger/Hilt, Room, and Moshi to KSP.
*   **Non-transitive R classes**: Improves incremental compilation. Set `android.nonTransitiveRClass=true`.

### 5. JVM Tuning
Allocate sufficient heap space for the Gradle daemon, especially for large multi-module projects.
*   **Action**: Set `org.gradle.jvmargs=-Xmx4g -XX:+UseParallelGC` in `gradle.properties`.

### 6. Lazy Task Configuration
Avoid using `tasks.create()`. Always use `tasks.register()` to ensure tasks are only configured when they are actually needed for the current build.

## Example Prompts

1.  "My Android Studio sync and build times are very slow. Can you review my `gradle.properties` and suggest optimizations?"
2.  "Help me migrate my project from kapt to KSP to improve compilation speed."
3.  "How do I use Gradle Build Scans to identify which tasks are taking the most time in my CI pipeline?"

## Expected Output

The user should receive:
*   A set of recommended `gradle.properties` settings.
*   A plan for migrating slow annotation processors (kapt) to KSP.
*   Instructions on how to generate and interpret a Gradle Build Scan (`--scan`).
*   Identification of eager task configurations that should be made lazy.
*   Advice on module structure to maximize parallel execution.

## Edge Cases & Common Mistakes

*   **Absolute Paths in Task Inputs**: Using absolute paths in task inputs or outputs breaks the build cache when shared across different machines (e.g., local vs CI). Always use `@PathSensitive(PathSensitivity.RELATIVE)`.
*   **Configuration-time I/O**: Performing file I/O or network requests during the configuration phase (e.g., reading a version from a file) blocks the entire build. Use Gradle's `ValueSource` API or `providers.fileContents()`.
*   **Eager Task Creation**: Using `tasks.create` or `allprojects { ... }` with eager logic slows down configuration as every module and task must be fully initialized even if not running.
*   **Large Monolithic Modules**: A single massive `:app` module prevents Gradle from parallelizing compilation. Break features into separate modules.
*   **Ignoring Build Scans**: Trying to optimize without measuring. Always use `./gradlew assembleDebug --scan` to get empirical data before making changes.

## Review Checklist

- [ ] Is `org.gradle.configuration-cache` enabled?
- [ ] Is `org.gradle.parallel` enabled?
- [ ] Are all dependencies using fixed versions (no `+`)?
- [ ] Has `kapt` been replaced by `ksp` where possible?
- [ ] Are tasks registered lazily using `tasks.register()`?
- [ ] Is `android.nonTransitiveRClass` enabled?
- [ ] Is the JVM heap (`-Xmx`) sized appropriately for the project?
- [ ] Are there any configuration-time file I/O operations?
- [ ] Is `android.builtInKotlin` utilized if on AGP 9.0+?
- [ ] Does the project use a remote build cache for CI?
