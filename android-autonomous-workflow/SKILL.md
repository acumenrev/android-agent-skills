---
name: android-autonomous-workflow
description: Orchestrate multi-step feature development from research and planning through robust implementation and testing in Android.
---

# Android Autonomous Workflow Expert

## Instructions
When tasked with complex feature development, use an autonomous workflow to decompose goals and systematically execute them.
1. **Research**: Analyze existing architecture (Hilt, Clean Architecture), packages, and AGP 9.0+ patterns.
2. **Plan**: Break the feature down into atomic, testable sub-tasks (e.g., Domain Model -> Data Source -> Repository -> ViewModel -> UI -> Tests).
3. **Implement**: Execute tasks adhering to strong modern defaults (Kotlin 2.2+, Jetpack Compose, Hilt, Coroutines/Flow).
4. **Verify**: Ensure the feature compiles, state management works as expected, and there are no UI regressions using Roborazzi or standard tests.

## Example Prompts
- "I need to build an offline-first task manager. Research the best approach, plan the Room schema using modern SSOT patterns, and implement the UI."
- "Orchestrate a workflow to add biometric authentication to the login screen using Hilt for state."
- "Decompose the goal of migrating our current state management to Flow across the 'profile' module."

## Expected Output
A sequence of verifiable steps or code blocks demonstrating incremental progress, starting with a clear architectural plan and ending with a fully functional, well-structured Android feature.

## Edge Cases & Common Mistakes
- **Skipping the Plan Phase**: Jumping straight into code and writing spaghetti logic. Always create a feature-first architectural plan first.
- **Ignoring Existing Architecture**: Overwriting or ignoring the project's existing modularization (e.g., mixing domain and data layers).
- **Failing to Verify**: Claiming success without ensuring Flow collectors are correctly scoped or that UI recompositions are optimized.

## Review Checklist
- [ ] Research phase documented and modern packages verified.
- [ ] Atomic tasks clearly defined.
- [ ] Implementation strictly follows the Hilt/Compose/Flow stack.
- [ ] Verification steps executed and passed.
