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

### Non-Interactive Mode (MANDATORY for AI Agents)

AI agents CANNOT interact with interactive prompts. **All commands MUST run non-interactively.** A stuck prompt = a stuck agent = wasted time.

#### Environment Variables (set before ANY npm/yarn/pnpm command)
```bash
export CI=true               # Signals non-interactive environment to most tools
export npm_config_yes=true   # npm: auto-accept all prompts
export YARN_ENABLE_IMMUTABLE_INSTALLS=false  # Yarn: don't fail on lockfile changes
```

#### npm / npx
```bash
# ✅ CORRECT — always use --yes for npx
npx -y create-vite@latest ./my-app -- --template vue-ts
npx -y create-next-app@latest ./my-app --ts --eslint --app --use-npm

# ✅ CORRECT — npm install (CI=true suppresses prompts)
CI=true npm install
CI=true npm ci              # Preferred for reproducible installs

# ❌ WRONG — interactive, WILL HANG
npx create-vite@latest my-app
npm init
npm create vite
```

#### Scaffolding Tools (create-vite, create-next-app, etc.)
**Every scaffolding tool has a non-interactive mode. You MUST find and use it.**

1. **Run `--help` first** to discover non-interactive flags before scaffolding
2. **Always pass all required options** (template, name, etc.) as CLI flags
3. **Common patterns:**
   - `create-vite`: `--template <template>` (must specify template to skip prompt)
   - `create-next-app`: `--ts --eslint --app --use-npm` (pass all choices as flags)
   - `create-react-app`: pass project name as argument
   - `npm init <initializer>`: use `npm create <initializer> -- <flags>` with `--yes` or template flags
4. **If a tool still prompts**, pipe `yes |` as last resort: `yes | npx -y create-tool ...`

#### pnpm
```bash
# ✅ CORRECT
pnpm install --no-frozen-lockfile
pnpm create vite my-app --template vue-ts

# ❌ WRONG — may prompt for lockfile confirmation
pnpm install
```

#### Yarn (v2+/Berry)
```bash
# ✅ CORRECT
yarn install --no-immutable
yarn create vite my-app --template vue-ts
```

#### General Rule
If **any** command could potentially prompt for user input:
1. Search for `--yes`, `--no-interactive`, `--non-interactive`, or `--batch` flag
2. Set `CI=true` in the environment
3. Pass all choices as explicit CLI flags (never rely on defaults that trigger prompts)
4. Test with `echo "" | command` if unsure whether it prompts

**Violation: running an interactive command that blocks the agent is a critical failure.**

### Checklist
- [ ] User input sanitized before commands
- [ ] Arguments as lists (no shell concat)
- [ ] Minimum permissions
- [ ] Exit codes checked
- [ ] Timeouts set
- [ ] stderr captured + logged
- [ ] Commands run in non-interactive mode (CI=true, --yes flags)
- [ ] Scaffolding tools invoked with all options as CLI flags

### Related
- Security Mandate GEMINI.md § Security Mandate
- Security Principles GEMINI.md § Security Principles
- Dependency Management @.gemini/skills/dependency-management-principles/SKILL.md