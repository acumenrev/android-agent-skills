---
name: android-testing
description: "Comprehensive testing strategy involving Unit, Integration, Hilt, and Screenshot tests for modern Android apps."
---

# Android Testing Expert

Expert guidance on testing modern Android applications using **JUnit 4/5**, **Kotlin Coroutines Test**, **Hilt**, and **Roborazzi** for screenshot testing.

## Instructions

Follow the testing pyramid to ensure a robust and maintainable application.

### 1. Unit Testing (ViewModels & Repositories)
Use `kotlinx-coroutines-test` to test business logic without an emulator. Use `TestDispatcher` to control time and execution.

```kotlin
@OptIn(ExperimentalCoroutinesApi::class)
class MyViewModelTest {
    private val testDispatcher = UnconfinedTestDispatcher()
    private lateinit var viewModel: MyViewModel

    @Before
    fun setup() {
        Dispatchers.setMain(testDispatcher)
        viewModel = MyViewModel(...)
    }

    @After
    fun tearDown() {
        Dispatchers.resetMain()
    }
}
```

### 2. Screenshot Testing (Roborazzi)
Verify UI correctness using **Roborazzi**. It runs on the JVM (Robolectric) for speed and doesn't require an emulator.

```kotlin
@RunWith(AndroidJUnit4::class)
@GraphicsMode(GraphicsMode.Mode.NATIVE)
@Config(sdk = [33], qualifiers = RobolectricDeviceQualifiers.Pixel5)
class MyScreenScreenshotTest {
    @get:Rule
    val composeTestRule = createAndroidComposeRule<ComponentActivity>()

    @Test
    fun captureMyScreen() {
        composeTestRule.setContent {
            MyTheme { MyScreen() }
        }
        composeTestRule.onRoot().captureRoboImage()
    }
}
```

### 3. Hilt Integration Tests
Use `HiltAndroidRule` to inject real or faked dependencies into your integration tests.

```kotlin
@HiltAndroidTest
class MyDaoTest {
    @get:Rule
    var hiltRule = HiltAndroidRule(this)

    @Inject
    lateinit var database: MyDatabase

    @Before
    fun init() {
        hiltRule.inject()
    }
}
```

### 4. Testing dependencies (`libs.versions.toml`)
Ensure you have the latest testing stack:
```toml
[libraries]
junit4 = { module = "junit:junit", version = "4.13.2" }
kotlinx-coroutines-test = { group = "org.jetbrains.kotlinx", name = "kotlinx-coroutines-test", version.ref = "kotlinxCoroutines" }
roborazzi = { group = "io. Takahirom.roborazzi", name = "roborazzi", version.ref = "roborazzi" }
hilt-android-testing = { group = "com.google.dagger", name = "hilt-android-testing", version.ref = "hilt" }
```

## Example Prompts

- "Write a unit test for a ViewModel that fetches data from a repository and updates a StateFlow, including error handling."
- "Show me how to set up a screenshot test for a Compose component using Roborazzi and a custom theme."
- "How do I mock a Retrofit API service in an integration test using MockWebServer and Hilt?"

## Expected Output

You should receive complete test classes with appropriate rules (JUnit, Hilt, Compose), setup/teardown logic, and clear test cases following the Arrange-Act-Assert pattern.

## Edge Cases & Common Mistakes

- **MainDispatcher in Tests**: Forgetting to set/reset the Main dispatcher (`Dispatchers.setMain`) in ViewModel tests will lead to `IllegalStateException` or tests hanging.
- **Race Conditions in StateFlow**: Testing `StateFlow` updates often requires `UnconfinedTestDispatcher` or careful use of `backgroundScope.launch` to ensure you don't miss emissions.
- **Robolectric Context**: Accessing `LocalContext.current` in screenshot tests without proper Robolectric configuration can cause crashes.
- **Leaking Databases**: Always close `Room` databases in `@After` to avoid memory leaks and inconsistent state between tests.

## Review Checklist

- [ ] Does the test use `kotlinx-coroutines-test` for coroutines?
- [ ] Are rules (`HiltAndroidRule`, `ComposeTestRule`) correctly ordered?
- [ ] Are tests descriptive and follow a naming convention (e.g., `whenX_thenReturnY`)?
- [ ] Are screenshot tests configured with a specific device qualifier (e.g., `Pixel5`)?
- [ ] Is the Main dispatcher reset after every test?
