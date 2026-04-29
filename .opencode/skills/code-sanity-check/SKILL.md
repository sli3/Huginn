# code-sanity-check

**name:** code-sanity-check  
**description:** Logic and syntax verification for Bash and C code.

---

## Sanity Check Protocol

### Trigger

Run when:

- User says `"sanity check"`, `"check this"`, or `"verify this script"`
- Before committing any new or heavily modified script
- After a Code session when the agent made structural changes

---

### Steps

1. **Syntax**  
   Run `shellcheck` on the script.  
   Fall back to `bash -n` if `shellcheck` is unavailable.
   ```bash
   shellcheck [script.sh] || bash -n [script.sh]
   ```

2. **Logic**  
   Check for:
   - Missing error handling (unchecked return codes, no `|| exit 1`)
   - Hardcoded paths that should be variables
   - Unquoted variables (e.g. `$VAR` instead of `"${VAR}"`)
   - Missing `set -euo pipefail` at the top

3. **Idempotency**  
   Verify the script can be run multiple times without side effects.  
   Look for destructive operations without guards (e.g. `rm -rf` without existence check).

4. **Fact Check**  
   Verify non-obvious command flags against `man` pages or `--help` output.  
   Flag any flags that look unfamiliar or have changed behaviour between versions.

5. **Report**  
   Summarise as one of:
   - ✅ **Passed** — no issues found
   - ⚠️ **Warnings** — issues found but not blocking; list each with explanation
   - ❌ **Failures** — blocking issues; list each and suggest fix

---

### Report Format

```
Sanity Check Report — [filename]
──────────────────────────────────
Syntax:       [Passed / Failed — reason]
Logic:        [Passed / Warnings — list]
Idempotency:  [Passed / Concern — description]
Fact Check:   [Passed / Flagged — description]

Result: ✅ Passed / ⚠️ Warnings / ❌ Failures
[Summary of any action required]
```

---

### Rules

- Never modify the file during a sanity check — report only
- If failures are found, do not proceed with commit until resolved
- Always run this before `git-workflow` on a Code session
