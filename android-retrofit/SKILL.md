---
name: android-retrofit
description: "Expert guidance on setting up and using Retrofit for type-safe HTTP networking in Android. Covers service definitions, coroutines, OkHttp configuration, and Hilt integration."
---

# Android Retrofit Networking Expert

Expert guidance on implementing type-safe network layers using **Retrofit**, **OkHttp**, and **Kotlin Coroutines**, following 2025 best practices.

## Instructions

When implementing network layers using **Retrofit**, follow these modern Android best practices.

### 1. Service Definition
Use `suspend` functions for all network calls. Prefer returning the deserialized body directly or `Response<T>` if you need access to headers or status codes.

```kotlin
interface GitHubService {
    @GET("users/{user}/repos")
    suspend fun listRepos(@Path("user") user: String): List<Repo>

    @GET("users/{user}")
    suspend fun getUserResponse(@Path("user") user: String): Response<User>
}
```

### 2. URL and Header Manipulation
*   **Dynamic Paths**: Use `@Path("id")`.
*   **Query Parameters**: Use `@Query("key")` or `@QueryMap`.
*   **Headers**: Use `@Header("Name")` for dynamic headers, or an OkHttp **Interceptor** for global headers (e.g., Auth tokens).

### 3. Hilt Integration
Provide your Retrofit instances as singletons in a Hilt module. Use `kotlinx.serialization` for JSON conversion.

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {

    @Provides
    @Singleton
    fun provideOkHttpClient(): OkHttpClient = OkHttpClient.Builder()
        .addInterceptor(HttpLoggingInterceptor().apply { level = HttpLoggingInterceptor.Level.BODY })
        .connectTimeout(30, TimeUnit.SECONDS)
        .build()

    @Provides
    @Singleton
    fun provideRetrofit(okHttpClient: OkHttpClient): Retrofit = Retrofit.Builder()
        .baseUrl("https://api.github.com/")
        .client(okHttpClient)
        .addConverterFactory(Json { ignoreUnknownKeys = true }.asConverterFactory("application/json".toMediaType()))
        .build()
}
```

### 4. Error Handling in Repositories
Always handle network exceptions in the Repository layer using `Result` or a similar wrapper to keep the UI state clean.

```kotlin
class GitHubRepository @Inject constructor(private val service: GitHubService) {
    suspend fun getRepos(username: String): Result<List<Repo>> = runCatching {
        service.listRepos(username)
    }.onFailure { exception ->
        // Log or handle specific exceptions (Timeout, No Internet)
    }
}
```

## Example Prompts

- "Create a Retrofit service for a weather API that supports dynamic city names and API keys in headers."
- "Show me how to set up an OkHttp Interceptor to automatically add an 'Authorization' bearer token to every Retrofit request."
- "How do I handle a 401 Unauthorized error globally across all Retrofit services to trigger a logout flow?"

## Expected Output

You should receive a complete networking implementation including the Retrofit interface, Hilt module for dependency injection, and a Repository class that handles network calls and error mapping.

## Edge Cases & Common Mistakes

- **Base URL Suffix**: Always ensure your `baseUrl` ends with a forward slash `/`. If it doesn't, relative paths starting with a slash will behave unexpectedly.
- **Leaking OkHttpClient**: Always provide `OkHttpClient` as a Singleton. Creating a new instance for every request is extremely expensive and can lead to connection pool exhaustion.
- **Main Thread Networking**: Retrofit handles thread switching internally when using `suspend` functions, but always ensure your UI doesn't block. The `suspend` function should be called from a `viewModelScope`.
- **Large Response Bodies**: Avoid using `Response.body()` multiple times as it can be consumed only once. If you need it more than once, store the result in a variable.

## Review Checklist

- [ ] Does the service use `suspend` functions?
- [ ] Is `kotlinx.serialization` used for JSON conversion?
- [ ] Are timeouts configured on `OkHttpClient`?
- [ ] Is a `HttpLoggingInterceptor` used (only in debug builds)?
- [ ] Are network calls wrapped in `runCatching` or `try-catch` in the Repository?
- [ ] Does the `baseUrl` end with `/`?
