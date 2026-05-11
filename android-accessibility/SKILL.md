---
name: android-accessibility
description: "Expert guidance for auditing and implementing Android accessibility, focusing on Jetpack Compose semantics, WCAG compliance, and inclusive design."
---

# Android Accessibility Expert

## Instructions

Analyze and implement accessibility in Android applications using Jetpack Compose, ensuring compliance with WCAG standards and providing a seamless experience for users of TalkBack, Switch Access, and other assistive technologies. Adhere to Kotlin 2.2+, AGP 9.0+, and modern Compose practices.

### 1. Semantics & Labeling
*   **Content Descriptions**: Provide meaningful `contentDescription` for all `Image`, `Icon`, and interactive elements. Use `null` for purely decorative elements. Descriptions should be context-aware (e.g., "Add to favorites" instead of "Heart icon").
*   **State Descriptions**: Use `Modifier.semantics { stateDescription = ... }` to communicate custom states (e.g., "In progress", "Verified") that aren't automatically handled by standard components.
*   **Roles**: Define the role of custom interactive components using `Modifier.semantics { role = Role.Button }` to help screen readers announce the correct element type.

### 2. Layout & Interaction
*   **Touch Targets**: Ensure all interactive elements have a minimum touch target size of **48x48dp**. Use `Modifier.padding` or `Box` wrappers to expand the hit area without changing the visual size.
*   **Merging Descendants**: Use `Modifier.semantics(mergeDescendants = true)` for complex UI elements (like list items or cards) to group related information into a single focusable unit.
*   **Headings**: Mark titles and section headers with `Modifier.semantics { heading() }` to enable navigation via headings in screen readers.

### 3. Visual & Navigational Accessibility
*   **Color Contrast**: Maintain a minimum contrast ratio of **4.5:1** for normal text and **3:1** for large text and essential icons.
*   **Focus Management**: Ensure a logical focus order (typically top-to-bottom, start-to-end). Use `FocusRequester` for manual focus management when necessary.
*   **Live Regions**: Use `Modifier.semantics { liveRegion = LiveRegionMode.Polite }` for dynamic content updates that should be announced immediately without interrupting the user.

## Example Prompts

*   "Audit this Jetpack Compose screen for accessibility violations, focusing on touch target sizes, content descriptions, and heading structure."
*   "Implement custom semantics for this custom circular progress indicator so it announces percentage updates to TalkBack as a live region."
*   "Refactor this complex Card component to use `mergeDescendants` so the entire card is announced as a single unit, and ensure the 'Favorite' button remains independently accessible if needed."

## Expected Output

*   A comprehensive accessibility audit report identifying specific issues and recommending fixes.
*   Production-ready Jetpack Compose code implementing modern semantics, logical focus handling, and inclusive design patterns.
*   Clear explanations of how the implementation benefits users of specific assistive technologies (e.g., TalkBack, Switch Access).

## Edge Cases & Common Mistakes

*   **Static Labels for Dynamic States**: Failing to update `contentDescription` or `stateDescription` when the state of a button changes (e.g., a "Mute" button that still says "Mute" after being pressed).
*   **Over-Labeling Decorative Elements**: Providing descriptions for every icon in a design, which creates "audio clutter" for screen reader users. Always use `null` for decorative assets.
*   **Invisible Focus Traps**: Creating custom overlays or dialogs that don't properly trap focus, allowing TalkBack users to navigate to elements "behind" the active UI layer.
*   **Relying Solely on Color**: Communicating important state or information (e.g., error vs. success) only via color without accompanying text or icons.

## Review Checklist

- [ ] All interactive elements have a minimum 48x48dp touch target.
- [ ] Content descriptions are meaningful, localized, and context-sensitive.
- [ ] Decorative elements have `contentDescription = null`.
- [ ] `mergeDescendants = true` is used on complex components to prevent fragmented announcements.
- [ ] Color contrast meets WCAG AA standards (4.5:1 minimum).
- [ ] Section headers are explicitly marked with `heading()`.
- [ ] Custom components have appropriate `Role` and `stateDescription` definitions.
