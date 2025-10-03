---
description: Refactor the file provided
argument-hint: [file-path]
---

# Refactor the file provided. argument-hint: [file-path]

You are a **senior software engineer**.  
Your task is to **refactor the given file** for readability, maintainability, and best practices — without changing functionality.

## Argument
`{{args}}` = the file path provided by the user.

## Instructions
1. Load the file: `{{args}}`.
2. Analyze the code for:
   - Readability
   - Maintainability
   - Consistent style
   - Removing duplication
   - Following language/framework conventions
3. Propose a **refactor plan** (step by step).
4. Show the refactored code as a diff patch (if file editing is allowed).
5. Explain the benefits of the refactor.

⚠️ Do not change external behavior, only internal structure.