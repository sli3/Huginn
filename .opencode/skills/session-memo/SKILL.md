# session-memo

**name:** session-memo  
**description:** Summarise the current session and save a timestamped memo to `.session-memos/` in the project root. Trigger on demand with "memo", "save session", or "end session".

---

## Session Memo Protocol

### Trigger

Run this skill when the user says any of:

- `"memo, this was a [Type] session"` — use the provided type directly
- `"memo"` — ask the user for session type before proceeding
- `"save session"` / `"summarise session"` / `"end session"`

If the session type is not stated, ask:

> "What type was this session? (Explore / Plan / Code / Review / Mixed)"

and wait for the answer before writing the memo.

---

### Steps

1. **Summarise** — Generate a concise session summary containing:
   - Session type (Explore / Plan / Code / Review)
   - What was discussed or decided (2–3 lines max)
   - What was changed or written (list files and functions touched)
   - What was NOT finished (next steps, max 3 bullet points)
   - Any important decisions or rejected approaches
   - Any mistakes made this session with their category

2. **Create memo directory:**
   ```bash
   mkdir -p .session-memos
   ```
   *(Safe to run even if directory exists)*

3. **Write memo file** — get timestamp with bash:
   ```bash
   TIMESTAMP=$(date +"%Y-%m-%d_%H-%M")
   MEMO_FILE=".session-memos/${TIMESTAMP}.md"
   ```
   Never overwrite existing memos — always fresh timestamp.

4. **Memo format:**

```markdown
# Session Memo — [YYYY-MM-DD HH:MM]

## Type
[Explore / Plan / Code / Review / Mixed]

## What We Did
- [2–3 concise bullet points]

## Functions Found *(Explore ONLY — MANDATORY)*
- `[function_name]`: [one line description]
- *(list every function — never group)*
- *(omit for non-Explore sessions)*

## Issues Found *(Explore ONLY — MANDATORY)*
- [exact issue] — Priority: High / Medium / Low
- *(list every issue — never omit or group)*
- *(omit for non-Explore sessions)*

## Agreed Next Step *(Explore ONLY — MANDATORY)*
[exactly what was decided to tackle next and why]
*(omit for non-Explore sessions)*

## Files Touched
- `[filename]`: [what changed]
- None *(if read-only session)*

## Decisions Made
- [approach chosen and why]
- [approach rejected and why]

## Mistakes Made
- [exact description] — Category: [Scope Creep / Safety Violation / Logic Error / Process Skip]
- None *(if no mistakes)*
  *(user may add to this manually after saving)*

## Not Finished
- [up to 3 clear next steps]

## Next Session Starter
[One specific actionable opening message for the next session]
```

5. **Confirm** — print file path only:
   ```
   Memo saved to .session-memos/[timestamp].md
   ```

---

### Rules

- Save to `.session-memos/` in project root only
- Never overwrite existing memos
- Keep summary under 300 words
- Next Session Starter must be specific and actionable
- Do **NOT** print full memo contents — just confirm file path
