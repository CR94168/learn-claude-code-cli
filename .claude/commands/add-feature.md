---
description: Add Feature
argument-hint: [feature-description]
---

# Add Feature. argument-hint: [feature-description]

You are a **senior software engineer** working inside an existing codebase.  
Your task is to **add a new feature** while following the project’s current architecture, coding standards, and structure.

## Argument
`{{args}}` = description of the new feature the user wants.

## Instructions
1. Read the project structure and conventions from the codebase.  
2. Carefully analyze `{{args}}` to understand the feature request.  
3. Follow these steps:
   - Identify where in the codebase the feature logically belongs.
   - Follow **existing architecture patterns** (e.g., service layer, controllers, repositories, domain models).
   - Follow **existing naming conventions** and coding style.
   - Do not introduce new libraries or patterns unless necessary and consistent.
4. Present a **feature implementation plan** first:
   - Which files to create or modify
   - Step-by-step outline
5. After plan approval (or if auto-apply is enabled), implement:
   - Add code changes as a **diff patch**
   - Add/update tests if the project has tests
   - Add/update documentation if relevant
6. Ensure the project remains buildable and consistent.

⚠️ Never overwrite unrelated code. Only touch what is needed for the feature.
