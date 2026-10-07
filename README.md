# ship-kit

A [Claude Code](https://code.claude.com) plugin that uses the right model for each part of the job.

```
/ship-kit:ship Double-clicking Pay applies the coupon twice
```

| Model | Role | What it does |
|---|---|---|
| **Haiku** | `scout` | Explores the codebase in parallel and reports back short summaries. Read-only. |
| **Sonnet** | main session | Plans, writes the code, and runs the tests. |
| **Opus** | `reviewer` | Reviews at 3 checkpoints. Read-only. |

Cheap, fast models do the reading, a solid model does the building, and the strongest model is only called when its judgment matters.

## Install

Run these two commands in your terminal:

```bash
claude plugin marketplace add denniscampos/ship-kit
claude plugin install ship-kit@ship-kit
```

Then start a **new** Claude Code session and type `/ship-kit:ship` to check it's there.

## Use it

Describe a feature or a bug in plain English:

```
/ship-kit:ship Add a "save card for later" checkbox at checkout
/ship-kit:ship Users get logged out when they refresh the settings page
```

That's it. It works out whether the task is a feature or a bug and takes it from there.

## What happens

```
/ship-kit:ship "…"
   │
   0. Classify    → FEATURE or BUG (asks you questions if it's vague)
   1. Scout       → 2–4 Haiku scouts explore in parallel
   2. Plan        → Sonnet writes a plan  →  ✅ Opus checkpoint A
   3. Build       → test first, then code  →  ✅ Opus checkpoint B (only if the same test fails twice)
   4. Verify      → screenshots that prove it works
   5. Finish      → Opus reviews the full diff and the screenshots  →  ✅ checkpoint C
   │
   Report         → what changed, test results, screenshots, and reviewer verdicts
```

It **never commits, pushes, or deploys**. You review the changes and commit them yourself.

### The 3 checkpoints

| Checkpoint | When | What Opus looks for |
|---|---|---|
| **A** | Before the plan is locked | Rules enforced in the wrong place, missed edge cases, schema or auth risks |
| **B** | When the same test fails twice | Whether it's fixing the root cause or just patching symptoms |
| **C** | Before calling it done | Hidden regressions, migrations that break existing data, weak screenshots |

### Screenshot proof

Every run has to finish with screenshots that prove the change works. They're saved outside your repo, in:

```
~/.claude/ship-evidence/<project>/<date>-<task>/
```

- **UI changes:** screenshots of the app working, plus a "before" screenshot for bugs when possible.
- **Backend-only changes:** a screenshot of the passing tests.

If it can't take a screenshot, it tells you why instead of claiming the work is verified.

## Good to know

- **Cost:** a run calls Opus up to 3 times. That uses more of your plan's limits than Sonnet alone, but much less than running Opus the whole time.
- **Small tasks:** for one-line changes it offers to skip the scouts and reviews.
- **Using the agents on their own:** you can ask for them directly, for example *"use the scout agent to find where coupons are applied"* or *"have the reviewer agent check my diff"*.
- **Screenshot tool:** it uses whatever is available. If nothing is, it falls back to `npx playwright screenshot`, which may install Playwright the first time.

## Update or remove

```bash
claude plugin update ship-kit@ship-kit      # get the latest version, then restart Claude Code
claude plugin uninstall ship-kit@ship-kit   # remove it
```

## Customize

Fork this repo and edit the Markdown files:

- `skills/ship/SKILL.md`: the step-by-step workflow
- `agents/scout.md`: the Haiku explorer
- `agents/reviewer.md`: the Opus reviewer

## License

MIT
