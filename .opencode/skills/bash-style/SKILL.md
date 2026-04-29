# bash-style

**name:** bash-style  
**description:** Bash scripting standards for this project. Apply these rules whenever writing or editing any `.sh` file.

---

## Bash Style Guide

### Trigger

Apply automatically whenever:

- Writing a new Bash script
- Editing an existing `.sh` file
- Reviewing a script in `code-sanity-check`

---

### Standards

#### Shebang
Always use the portable form:
```bash
#!/usr/bin/env bash
```
Never use `#!/bin/bash` — not portable across all systems.

---

#### Error Handling
Always include at the top of every script, immediately after the shebang:
```bash
set -euo pipefail
```
- `-e` — exit immediately on error
- `-u` — treat unset variables as errors
- `-o pipefail` — catch errors in pipelines

---

#### Variables
Always quote all variable expansions:
```bash
# Correct
"${VAR}"
"${MY_ARRAY[@]}"

# Wrong
$VAR
${VAR}   # unquoted
```
Use `UPPER_SNAKE_CASE` for environment variables and script-level constants.  
Use `lower_snake_case` for local function variables.

---

#### Functions
- Use `snake_case` names
- Add a comment above each function describing its purpose
- Always declare internal variables with `local`

```bash
# Rotate logs older than N days
rotate_logs() {
  local log_dir="${1}"
  local max_days="${2}"
  find "${log_dir}" -name "*.log" -mtime +"${max_days}" -delete
}
```

---

#### Output
- Use `echo` for user-facing messages
- Use `>&2` to redirect errors to stderr

```bash
echo "Starting backup..."
echo "Error: directory not found" >&2
```

---

#### Conditionals
Prefer `[[ ]]` over `[ ]` for all conditionals in Bash scripts:
```bash
# Correct
if [[ -f "${config_file}" ]]; then

# Avoid
if [ -f "$config_file" ]; then
```

---

#### Cleanup & Traps
For scripts that create temporary files, always register a trap:
```bash
TMPFILE=$(mktemp)
trap 'rm -f "${TMPFILE}"' EXIT
```

---

#### Script Header Template
Every new script should begin with:

```bash
#!/usr/bin/env bash
# ──────────────────────────────────────────────
# [Script Name] — [One line description]
# Usage: ./[script-name].sh [args]
# ──────────────────────────────────────────────
set -euo pipefail
```

---

### Rules

- Never write a script without `set -euo pipefail`
- Never leave variables unquoted
- Never use `#!/bin/bash`
- Always use `local` for function variables
- Always add a comment above every function
