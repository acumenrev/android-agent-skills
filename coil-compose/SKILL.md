---
name: coil-compose
description: "Expert guidance on using Coil for image loading in Jetpack Compose. Covers AsyncImage, performance optimization, and custom image requests."
---

# Coil Image Loading Expert

Expert guidance on implementing efficient image loading in Jetpack Compose using **Coil** (Coroutines Image Loader), following modern performance and UX standards.

## Instructions

When implementing image loading in Jetpack Compose, use **Coil**. It is optimized for Compose and provides a seamless developer experience.

### 1. Primary Composable: `AsyncImage`
Use `AsyncImage` for almost all use cases. It handles size resolution automatically and integrates with Compose's layout system.

```kotlin
AsyncImage(
    model = ImageRequest.Builder(LocalContext.current)
        .data("https://example.com/image.jpg")
        .crossfade(true)
        .build(),
    placeholder = painterResource(R.drawable.placeholder),
    error = painterResource(R.drawable.error),
    contentDescription = stringResource(R.string.description),
    contentScale = ContentScale.Crop,
    modifier = Modifier.size(100.dp).clip(CircleShape)
)
```

### 2. Low-Level Control: `rememberAsyncImagePainter`
Use `rememberAsyncImagePainter` only when you need a `Painter` for non-composable targets like `Canvas`, or for legacy interoperability. Note that it does not detect size automatically.

### 3. Loading & Error States: `SubcomposeAsyncImage`
Use `SubcomposeAsyncImage` if you need complex custom UI for loading or error states (e.g., a custom spinner or a retry button).

> [!WARNING]
> Subcomposition is expensive. Avoid using `SubcomposeAsyncImage` inside `LazyColumn` or `LazyRow` to prevent frame drops.

### 4. Performance & UX Best Practices
*   **Crossfade**: Always use `.crossfade(true)` for a smooth transition from placeholder to image.
*   **ContentScale**: Use `ContentScale.Crop` or `ContentScale.Fit` to ensure images are scaled correctly for their container.
*   **Content Description**: Always provide a meaningful `contentDescription` for accessibility, or `null` if the image is purely decorative.
*   **Singleton ImageLoader**: Configure a singleton `ImageLoader` in your `Application` class or via Hilt to share memory and disk caches across the app.

## Example Prompts

- "Help me implement a circular profile image loader using Coil that shows a generic avatar placeholder while loading."
- "How do I optimize Coil image loading for a horizontal scrolling list of 100 high-resolution images to avoid jank?"
- "Show me how to use Coil to load a local image from the res/drawable folder with a custom transformation like Blur."

## Expected Output

You should receive concise Compose code using `AsyncImage`, including a properly configured `ImageRequest` with crossfade, placeholders, and error handling, along with advice on performance if applicable.

## Edge Cases & Common Mistakes

- **Subcompose in Lists**: Using `SubcomposeAsyncImage` in a `LazyColumn` is a common cause of scrolling performance issues. Use a simple `placeholder` in `AsyncImage` instead.
- **Missing contentDescription**: Failing to provide a content description makes your app inaccessible to screen readers.
- **Large Image Overhead**: Loading original-size images into small containers. Coil's `AsyncImage` handles this by default, but ensure you don't override it with `Size.ORIGINAL` unless necessary.
- **Dynamic URLs**: Forgetting to use a `remember` block or a stable key if the URL changes frequently in a way that triggers unnecessary recompositions.

## Review Checklist

- [ ] Is `AsyncImage` used instead of `rememberAsyncImagePainter` where possible?
- [ ] Is `crossfade(true)` enabled for better UX?
- [ ] Is a `contentDescription` provided or explicitly set to `null`?
- [ ] Is `SubcomposeAsyncImage` avoided in performance-critical lists?
- [ ] Are placeholders and error images provided?
- [ ] Is the `modifier` correctly applied for sizing and clipping?
