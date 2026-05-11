# GEMINI.md

This file provides guidance to Gemini CLI when working with code in this repository.

## Communication Style

- **No AI fluff.** Skip phrases like "Thanks for the thoughtful feedback!", "Happy to help!", "I appreciate...", etc.
- **No soliciting opinions.** Don't end responses with "Would you like me to...", "Let me know if...", "What do you think?", etc.
- **Direct and terse.** State facts, provide options when relevant, then stop.

## What this repo is

This repository is a maintained collection of Agent Skills for modern Android development, centered on **AGP 9.0+**, **Kotlin 2.2+**, and **Jetpack Compose**.

This is not a buildable app or Android project. The main work here is **authoring, refining, and reorganizing skill content** so the skills are accurate, discoverable, and useful to future agents.

## Core repo shape

Treat this as a content + metadata repository:
- `<skill>/SKILL.md` — canonical skill instructions
- `<skill>/references/*.md` — deeper examples, recipes, and reference material
- `README.md` — public catalog/distribution document

There is no normal app-style build or test loop here. Validation is mostly editorial and structural.

## Repo-specific authoring rules

- Keep `SKILL.md` concise. Push long examples and deep detail into local `references/` files.
- Use progressive disclosure. Important reference files should be linked directly from `SKILL.md`.
- Keep skills self-contained. A skill may mention a sibling skill, but it should not depend on another skill's files.
- Preserve topic boundaries between sibling skills instead of recombining broad topics.
- **Strong Modern Defaults:** Prefer one strong modern default over presenting many competing approaches. For Android, this stack is:
  - **Language**: Kotlin 2.2+ (with K2 compiler)
  - **UI**: Jetpack Compose (Material 3)
  - **Architecture**: Clean Architecture / MVVM / MVI
  - **Dependency Injection**: Hilt
  - **Concurrency**: Kotlin Coroutines & Flow
  - **Networking**: Retrofit + Kotlinx Serialization
  - **Persistence**: Room
- Keep guidance focused on current Android APIs and AGP 9.0 features (like built-in Kotlin). Avoid deprecated patterns unless the skill is explicitly about migration.
- When a skill changes substantially, sanity-check it with realistic prompts to make sure the description, scope, and guidance still work well in practice.

## Ground Android claims in official sources

This repo is about Android frameworks and platform behavior, so skill content should be grounded in official Android sources whenever possible.

- Prefer Android Developer documentation, Material Design guidelines, and Google I/O transcripts over memory, blog posts, or generic web summaries.
- Re-check official docs when updating API names, availability, behavior, or best-practice recommendations.

## Skill metadata matters

Frontmatter on each `SKILL.md` is important for discovery and routing.

- Keep `name` aligned with the folder name.
- Make `description` specific about what the skill covers, which frameworks/APIs it targets, and when it should be used.
- Favor descriptions that help an agent choose the right skill among nearby alternatives.

## Structure patterns to preserve

Most skills in this repo follow a stable pattern. Preserve it unless there is a strong reason not to:
- clear opening scope statement
- YAML frontmatter with `name` and `description`
- `## Instructions`
- `## Example Prompts`
- `## Expected Output`
- `## Edge Cases & Common Mistakes`
- `## Review Checklist`
