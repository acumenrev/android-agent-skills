---
name: android-architecture
description: "Expert guidance on designing and implementing modern Android application architecture using Clean Architecture, Hilt, and modularization strategies."
---

# Android Architecture Expert

## Instructions

Design and implement robust, scalable, and testable Android applications following the **Guide to App Architecture** and **Clean Architecture** principles. Prioritize Kotlin 2.2+, AGP 9.0+, Jetpack Compose, and Hilt.

### 1. Layered Architecture (U-D-D)
Adhere to a strict separation of concerns across three primary layers. Dependencies must always flow **inwards** towards the domain logic.

*   **UI Layer (Presentation)**:
    *   **Responsibility**: Managing UI state and handling user interactions.
    *   **Tech Stack**: Jetpack Compose, ViewModels, `StateFlow`.
    *   **Rule**: ViewModels should only depend on Use Cases (Domain) or Repositories (Data).
*   **Domain Layer (Business Logic)**:
    *   **Responsibility**: Encapsulating complex business rules into reusable Use Cases (e.g., `GetLatestNewsUseCase`).
    *   **Requirement**: This layer must be **pure Kotlin/Java** with zero Android framework dependencies (no `android.*` imports).
    *   **Components**: Use Cases, Domain Models, Repository Interfaces.
*   **Data Layer**:
    *   **Responsibility**: Handling data operations (fetching, caching, persistence).
    *   **Tech Stack**: Room (local), Retrofit (remote), DataStore (preferences).
    *   **Components**: Repository implementations, Data Sources, Mappers (Data -> Domain).

### 2. Dependency Injection with Hilt
Use Hilt as the standard for dependency injection to ensure modularity and ease of testing.

*   **Scoping**: Use `@Singleton` for app-wide services (Network, DB) and `@ViewModelScoped` for ViewModel-specific dependencies.
*   **Interface Binding**: Use `@Binds` in abstract modules to bind interface implementations to their abstractions.
*   **Constructor Injection**: Prefer constructor injection over field injection whenever possible.

### 3. Modularization Strategy
For production-grade apps, use a multi-module strategy to improve build performance and enforce architectural boundaries.

*   **Feature Modules**: `:feature:[name]` modules containing UI and ViewModels for a specific vertical slice.
*   **Core Modules**:
    *   `:core:ui`: Common design system components and themes.
    *   `:core:data`: Centralized data management (Retrofit, Room).
    *   `:core:domain`: Shared business logic and models.
    *   `:core:common`: Utilities and cross-cutting concerns.
*   **App Module**: `:app` serves as the orchestrator, initializing Hilt and connecting all modules.

## Example Prompts

*   "Propose a multi-module project structure for a large-scale e-commerce Android app using Clean Architecture and Hilt, explaining the responsibility of each module."
*   "Refactor this monolithic `:app` module into feature-based modules, ensuring strict dependency boundaries between the UI and Data layers."
*   "Implement a robust offline-first synchronization strategy using Room, Retrofit, and WorkManager within a Repository pattern, following Clean Architecture principles."

## Expected Output

*   A structured architectural blueprint or project layout.
*   Production-ready boilerplate for Hilt modules, Repositories, and Use Cases.
*   Clear documentation on data flow, state management, and dependency directions.

## Edge Cases & Common Mistakes

*   **Circular Dependencies**: Poorly defined module boundaries that lead to modules depending on each other, causing build failures. Always use an aggregator module or a `:core:common` module to break cycles.
*   **Leaking Android Framework into Domain**: Importing `android.content.Context` or other Android-specific classes into Use Cases or Domain models. This breaks unit testability and pure Kotlin principles.
*   **God Repositories**: Creating a single `MainRepository` that handles everything. Instead, split repositories by data source or domain (e.g., `UserRepository`, `ProductRepository`).
*   **Over-modularization**: Creating too many tiny modules for a simple application, leading to significant build configuration overhead without tangible benefits.

## Review Checklist

- [ ] Domain layer contains zero `android.*` imports and is pure Kotlin.
- [ ] Dependencies flow strictly Inwards: UI -> Domain <- Data.
- [ ] Every Repository implementation provides a main-safe `suspend` API.
- [ ] Hilt is used for all dependency injection, with appropriate scoping.
- [ ] ViewModels expose state via `StateFlow` and handle events via `ViewModelScope`.
- [ ] Data sources (Room, Retrofit) are abstracted behind Repository interfaces.
- [ ] Multi-module boundaries are logical and prevent circular dependencies.
