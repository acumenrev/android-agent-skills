---
name: android-data-layer
description: "Expert guidance on implementing a robust, offline-first Android data layer using the Repository pattern, Room, Retrofit, and DataStore."
---

# Android Data Layer Expert

## Instructions

Implement a reliable, scalable, and offline-first data layer in Android. Coordinate between multiple data sources while maintaining a single source of truth (SSOT). Prioritize Kotlin 2.2+, AGP 9.0+, Room, Retrofit, and Hilt.

### 1. Repository Pattern & SSOT
*   **Single Source of Truth**: Design repositories to prioritize local storage (Room) as the source of truth for the UI. The UI observes local data, and the repository handles background synchronization.
*   **Main-Safety**: Ensure all repository methods are main-safe by using `withContext(Dispatchers.IO)` internally.
*   **Abstraction**: Use interfaces for repositories to decouple the domain layer from data implementation details, facilitating easier unit testing with mocks.

### 2. Local Persistence (Room & DataStore)
*   **Reactive Queries**: Return `Flow<T>` or `PagingSource` from Room DAOs to ensure the UI updates automatically when the database changes.
*   **DataStore**: Use `DataStore` (Preferences or Proto) for small key-value pairs or settings, replacing the deprecated `SharedPreferences`.
*   **Migrations**: Always provide explicit `Migration` paths for Room database changes to prevent data loss or crashes during app updates.

### 3. Remote Data (Retrofit)
*   **Suspend Functions**: Define Retrofit interface methods as `suspend` functions.
*   **Mapping**: Convert remote DTOs (Data Transfer Objects) into clean Domain entities immediately within the data layer to prevent API implementation details from leaking into the UI or Domain layers.
*   **Error Handling**: Use a `Result` wrapper or explicit exception handling to manage network errors, timeouts, and API-specific error codes gracefully.

### 4. Offline-First Synchronization
*   **Sync Strategies**: Implement "Stale-While-Revalidate" (show local data immediately while refreshing in the background) or "Offline-First" (write to local DB first, then sync to server).
*   **WorkManager**: Use `WorkManager` for reliable, persistent background synchronization tasks that must complete even if the app is closed or the device restarts.

## Example Prompts

*   "Implement an offline-first repository for a Task Management app using Room for persistence and Retrofit for remote sync, following Clean Architecture."
*   "Design a data synchronization strategy using WorkManager that handles network retries and exponential backoff for a multi-module news application."
*   "Create a data mapping layer that converts complex Retrofit API response models into clean Domain entities, ensuring no nullability issues or leaky abstractions."

## Expected Output

*   A robust, thread-safe data layer implementation with clear separation between local and remote sources.
*   Reactive Room DAOs and Retrofit services integrated into a Single Source of Truth repository.
*   Production-ready boilerplate for data entities, DTOs, and mapping logic.

## Edge Cases & Common Mistakes

*   **Leaky Abstractions**: Passing Retrofit-specific classes (like `Response<T>`) or Room entities directly to the UI layer. Always map to Domain models.
*   **Ignoring Network Errors**: Failing to handle `IOException` or `HttpException` during network calls, leading to unhandled crashes or "silent" failures in the UI.
*   **Database on Main Thread**: Forgetting to use `suspend` or background dispatchers for Room operations, which causes UI stutters or ANRs (Application Not Responding).
*   **Race Conditions in Sync**: Improperly handling concurrent writes to the local database and remote server, leading to data inconsistency. Use transactions and proper conflict resolution strategies.

## Review Checklist

- [ ] Repository exposes data via `Flow` from the local database (SSOT).
- [ ] Remote data is mapped to Domain models before reaching the Domain/UI layers.
- [ ] All repository methods are main-safe.
- [ ] Room DAOs use `suspend` for one-shot writes and `Flow` for observable reads.
- [ ] Error handling covers network timeouts, lack of connectivity, and API errors.
- [ ] `DataStore` is used instead of `SharedPreferences`.
- [ ] WorkManager is used for critical background synchronization.
- [ ] Database migrations are defined for all schema changes.
