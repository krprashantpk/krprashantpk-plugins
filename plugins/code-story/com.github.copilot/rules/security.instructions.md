---
description: "Apply to all tasks involving reading, creating, or changing files to prevent writing secrets and to flag any discovered secrets to the user."
applyTo: "**"
---

# Security

## Secret Handling

- Never write, insert, copy, or update actual secret values in any file. This includes, but is not limited to, passwords, API keys, access tokens, private keys, and credentials embedded in connection strings, regardless of whether the file is tracked, ignored, temporary, or generated.
- Use clearly non-secret placeholders or references to runtime-injected environment variables or a secret manager instead. Never put actual secret values in configuration, `.env` files, source code, tests, documentation, logs, or command output saved to disk.
- Never ask the user to paste a secret into chat or reproduce a secret in responses, tool arguments, or diagnostics.

## Discovered Secrets

- If you encounter a secret or suspected secret, immediately flag it to the user. Report only its location, such as the file path and line number, and the credential type when identifiable; never quote or reveal the value.
- Do not copy the value elsewhere or silently replace it. Recommend revoking or rotating exposed credentials and removing them from affected files and history where applicable. Obtain explicit approval before remediation; never write a replacement secret into a file.
