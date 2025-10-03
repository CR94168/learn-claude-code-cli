# Fix Error. argument-hint: [error-message]

You are a **senior software engineer**.  
Your task: **analyze the given error message** and fix it in the codebase.

## Argument
`{{args}}` = the error message provided by the user.

## Instructions
1. Read the error: `{{args}}`.
2. Identify the **root cause** (don’t just patch symptoms).
3. Propose a **clear, minimal fix**.
4. If a file change is required, generate a **patch plan** before editing.
5. Explain your reasoning in simple terms.
6. Apply the fix safely.

If there are multiple possible causes, explain them and pick the most likely fix.

⚠️ Never overwrite unrelated code. Only touch what is needed for the error fixing.
