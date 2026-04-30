---
name: git-workflow
description: Secure Git workflow with review gates and UK English commit standards. Triggers on "commit", "git commit", "push", or "save to git". Always run after code-sanity-check passes.
---

## Secure Git Workflow

### Steps

1. **Stage Check**  
   Run `git status` to confirm only intended files are staged.
   ```bash
   git status
   ```
   Report which files are staged, modified, or untracked.  
   If unexpected files are staged, stop and ask the user to review.

2. **Review Gate**  
   Run `git diff --cached` and show the full output.
   ```bash
   git diff --cached
   ```
   Wait for explicit **"OK"** before proceeding.  
   Do not continue if the user does not confirm.

3. **Commit Format**  
   Use UK English in all commit messages.  
   Format: `[category]: [description in past tense]`

   | Category | Use for |
   |----------|---------|
   | `feat` | New feature added |
   | `fix` | Bug corrected |
   | `refactor` | Code restructured, no behaviour change |
   | `docs` | Documentation updated |
   | `chore` | Build, config, or tooling changes |
   | `test` | Tests added or updated |

   **Examples:**
   ```
   fix: corrected log rotation path
   feat: added apt cleanup stage
   docs: updated README with setup steps
   refactor: extracted validation into separate function
   ```

4. **Push Gate**  
   Ask explicitly:
   > "Shall I push to `main`? (Yes / No)"

   Never push without an explicit **"Yes"**.  
   If the user says No, stop and confirm what should happen instead.

5. **Safety**  
   Never use `--force` or `-f` under any circumstance.  
   If a force push appears necessary, stop and escalate to the user with an explanation.

---

### Rules

- Never skip the diff review gate
- Never push without explicit approval
- Never use `--force` or `-f`
- Always use UK English in commit messages
- One logical change per commit — do not bundle unrelated changes
