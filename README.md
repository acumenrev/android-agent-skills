# Android Agent Skills — Optimized for Gemini CLI

[![Kotlin](https://img.shields.io/badge/Kotlin-2.2-7F52FF?logo=kotlin&logoColor=white)](https://kotlinlang.org)
[![Platform](https://img.shields.io/badge/Platform-Android-3DDC84.svg?logo=android&logoColor=white)](https://developer.android.com)
[![Gemini CLI](https://img.shields.io/badge/Gemini%20CLI-compatible-blue.svg)](https://github.com/google/gemini-cli)

A collection of 18 agent skills for modern Android development (AGP 9.0+, Kotlin 2.2, Jetpack Compose), ported and optimized specifically for **Gemini CLI**.

Every skill has been normalized for Gemini CLI compatibility:
- **Normalized Metadata:** `SKILL.md` files use single-line, quoted descriptions and minimal YAML frontmatter for efficient discovery.
- **Security Audited:** Reviewed for shell injection and unsafe command execution patterns. Includes a robust set of emulator automation scripts.
- **AGP 9.0 Optimized:** Build-related skills have been updated to reflect the latest AGP 9.0 standards, including built-in Kotlin support and new DSL patterns.
- **Agent Guidance:** Includes a root-level `GEMINI.md` to provide global instructions and authoring rules to the agent.

## Install

To use these skills with Gemini CLI, you can install them from this local directory.

### Install All Skills

Install every skill in the collection to your user-level skill store:

```bash
for d in */; do
  if [ -f "$d/SKILL.md" ]; then
    gemini skills install "$d" --scope user
  fi
done
```

### Install Specific Skills

Install individual skills as needed:

```bash
gemini skills install ./android-architecture --scope workspace
gemini skills install ./compose-ui --scope user
```

### Reload Skills

After installation, reload your session to enable the new skills:

```
/skills reload
```

## Global Context

This repository includes a `GEMINI.md` file. When you work with Gemini CLI in this directory, the agent automatically reads this file to understand the project's architecture, communication style, and Android-specific documentation rules.

## Skills Catalog

### Architecture & Foundation
- **android-architecture**: Clean architecture, UDF, and project structure guidance.
- **android-viewmodel**: Proper implementation using `StateFlow` and `SharedFlow`.
- **android-data-layer**: Repository pattern, Room, and data source abstraction.
- **kotlin-concurrency-expert**: Deep dive into Kotlin Coroutines and Threading.
- **android-coroutines**: Best practices for Coroutines execution and scopes.

### UI & UX
- **compose-ui**: Jetpack Compose best practices, state hoisting, and modifiers.
- **compose-navigation**: Type-safe navigation in Compose.
- **coil-compose**: Efficient image loading with Coil in Compose.
- **android-accessibility**: Semantic trees, accessibility roles, and inclusive design.

### Build & Performance
- **android-gradle-logic**: AGP 9.0+ build logic, convention plugins, and version catalogs.
- **gradle-build-performance**: Optimizing build times, configuration cache, and build scans.
- **compose-performance-audit**: Diagnosing and fixing recomposition issues and jank.

### Testing & Automation
- **android-testing**: Unit testing (JUnit5), Hilt integration, and Roborazzi screenshot testing.
- **android-emulator-skill**: A suite of Python/Shell scripts for accessibility-driven emulator automation.

### Networking & Integration
- **android-retrofit**: Type-safe HTTP client configuration and error handling.

### Migration
- **xml-to-compose-migration**: Systematic workflow for porting legacy XML views to Compose.
- **rxjava-to-coroutines-migration**: Refactoring reactive streams to structured concurrency.

## License

MIT License. This project is not affiliated with, endorsed by, or sponsored by Google LLC.
