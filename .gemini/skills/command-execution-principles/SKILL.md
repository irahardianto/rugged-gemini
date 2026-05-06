---
name: command-execution-principles
description: >-
  Safe command execution: input sanitization, timeout handling, output capture,
  error propagation. For spawning processes, shell commands, system calls.
user-invocable: false
---

## Command Execution Principles

### Security
- ❌ `exec(userInput)`, `shell("rm " + userFile)`
- ✅ Argument lists (no shell string concat), validate/sanitize all args
- Never run as root/admin without explicit human approval. Elevated perms required → STOP and request auth.
- Least-privilege service accounts

### Portability
- Use language stdlib over shell commands (file I/O APIs instead of cat/cp/mv)
- Test on all target OS (path joining functions, not / concatenation)

### Error Handling
- Check exit codes (non-zero = failure)
- Capture + log stderr
- Set timeouts for long-running
- Handle "command not found" gracefully

### Checklist
- [ ] User input sanitized before commands
- [ ] Arguments as lists (no shell concat)
- [ ] Minimum permissions
- [ ] Exit codes checked
- [ ] Timeouts set
- [ ] stderr captured + logged

### Related
- Security Mandate GEMINI.md § Security Mandate
- Security Principles GEMINI.md § Security Principles