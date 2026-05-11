---
name: android-emulator-skill
description: "Expert guidance on managing Android Virtual Devices (AVDs), automating UI interactions, and performing build/test operations using CLI tools and scripts."
---

# Android Emulator & Automation Expert

## Instructions

Manage Android emulators and automate app interactions using accessibility-driven navigation and structured CLI tools. Focus on reliability, token efficiency, and semantic interaction rather than coordinate-based tapping.

### 1. Emulator Lifecycle Management
*   **Health Checks**: Use `emu_health_check.sh` (or `.ps1`) to verify the environment (ADB, SDK, Java, Gradle).
*   **AVD Management**: Use `emulator_manage.py` to list, boot, and shutdown emulators programmatically.
*   **App Lifecycle**: Use `app_launcher.py` to install, launch, terminate, or uninstall applications.

### 2. Semantic UI Interaction
*   **Screen Mapping**: Use `screen_mapper.py` to analyze the current UI hierarchy and identify interactive elements (buttons, text fields) via `uiautomator`.
*   **Navigator**: Use `navigator.py` to interact with elements by text, resource-id, or type. Always prefer semantic identifiers over pixel coordinates.
*   **Gestures & Keyboard**: Use `gesture.py` for swipes and scrolls, and `keyboard.py` for text entry and hardware button events (Back, Home, Enter).

### 3. Build & Diagnostics
*   **Build Automation**: Use `build_and_test.py` to trigger Gradle tasks (assemble, install, connectedCheck) and parse results.
*   **Log Monitoring**: Use `log_monitor.py` for intelligent `logcat` filtering by package name, tag, or priority level.
*   **JSON Output**: Use the `--json` flag on all scripts for machine-readable data integration.

## Example Prompts

*   "Automate the login flow for `com.example.app` using `navigator.py`: enter 'test@example.com' in the email field, 'password123' in the password field, and click the 'Login' button."
*   "Perform a UI audit of the current screen by mapping all interactive elements and verifying that every 'Button' has a corresponding 'content-desc' for accessibility."
*   "Monitor the system logs for `com.example.app` with 'Error' priority for the next 60 seconds and report any stack traces found in JSON format."

## Expected Output

*   Reliable automation of app flows and emulator management tasks.
*   Structured UI maps and filtered log data for debugging and testing.
*   Clear diagnostic reports on environment health and build/test outcomes.

## Edge Cases & Common Mistakes

*   **Timing & Race Conditions**: Attempting to interact with a UI element before it is fully rendered. Use retries or wait-for-element logic in scripts.
*   **Multiple Devices**: Forgetting to specify a device serial (`-s <serial>`) when more than one emulator or physical device is connected via ADB.
*   **Stale UI Hierarchy**: Relying on a screen map that was captured before a screen transition or dynamic content update. Always refresh the map before critical interactions.
*   **Emulator Resource Exhaustion**: Running multiple emulators without sufficient system memory or disk space, leading to slow boot times or frequent crashes.

## Review Checklist

- [ ] Emulator environment passes health checks.
- [ ] UI interactions use semantic identifiers (text, ID) instead of coordinates.
- [ ] Log monitoring is scoped to the specific package and priority level.
- [ ] Automation scripts include error handling for missing elements or timeouts.
- [ ] Build tasks are correctly mapped to Gradle commands.
- [ ] JSON output is used for integration with other tools or agents.
- [ ] `adb` server is responsive and device serials are correctly targeted.
