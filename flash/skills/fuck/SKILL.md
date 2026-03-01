---
name: fuck
description: Use when the user says "fuck", "you do it", "no u", "just do it", "run it yourself", "do it for me", or invokes /fuck - extracts commands from the previous assistant message that were suggested for the user to run manually (e.g. interactive auth, privileged operations) and executes them directly.
---

# Fuck

Run the command Claude just told you to run manually.

## How It Works

1. Look at the **most recent assistant message** in the conversation
2. Find commands that were suggested for the user to run in their terminal — these are typically in code blocks or inline code
3. Identify which commands were deferred because they needed interactive input, elevated privileges, or browser-based auth
4. Execute them using Bash with `dangerouslyDisableSandbox: true` to bypass sandbox restrictions

## Identifying the Right Commands

Look for these patterns in the previous assistant message:
- Code blocks containing shell commands
- Phrases like "run this in your terminal", "you need to run", "run that", "execute this"
- Commands preceded by context about auth failures, permission issues, or interactive requirements
- Commands that Claude explicitly said it *couldn't* run itself

If multiple commands were suggested, run them in the order they appeared.

## Execution

- Run each identified command with `dangerouslyDisableSandbox: true`
- Use a generous timeout (120000ms+) since interactive commands may take time
- After execution, report the result and continue with whatever Claude said it would do next

## Example

Previous assistant message:
> Your GCP auth tokens have expired. You need to re-authenticate:
> ```
> gcloud auth application-default login
> ```
> Run that in your terminal, then I'll proceed with the terraform apply.

Action: Execute `gcloud auth application-default login` with dangerouslyDisableSandbox, then proceed with the terraform apply.
