---
name: ship
description: Build a feature or fix a bug end to end. Haiku scouts explore, Sonnet plans and implements, Opus reviews at fixed checkpoints. Stops before committing.
argument-hint: [feature or bug description]
disable-model-invocation: true
model: sonnet
effort: high
---
Task: $ARGUMENTS

This skill uses two agents from the ship-kit plugin: `scout` and `reviewer`. They may be listed as `ship-kit:scout` and `ship-kit:reviewer`; use whichever name is available.

## 0. Classify
Decide whether this is a FEATURE or a BUG and state it in one line.
If the task is trivial (one file, obvious change), say so and ask me whether to skip the scouts and checkpoints.
If the task is ambiguous, ask me up to 3 clarifying questions before scouting.

## 1. Scout
Launch 2–4 `scout` subagents in parallel, in a single message. Do not explore the codebase broadly yourself.
- FEATURE: find similar existing features to copy patterns from, the files the feature will touch, existing tests for that area, and relevant library/API docs.
- BUG: find every code path involved in the symptom, existing tests for that area, and recent changes to those files (`git log`).

## 2. Plan → Checkpoint A (reviewer)
Write a short plan:
- FEATURE: acceptance criteria (what "done" looks like), files to change, approach, and the tests that prove it.
- BUG: the suspected root cause, files to change, the fix, and the test that reproduces the bug.
- Both: invariants that must hold (auth, data integrity, schema contracts, concurrency).

Send the `reviewer` subagent this brief:
- the task and its classification
- the plan
- the scout findings, with file paths

Apply every BLOCKING item. If you disagree with one, tell me why instead of silently ignoring it.
Show me the final plan in a few lines, then continue without waiting.

## 3. Build → Checkpoint B (reviewer, only if stuck)
- BUG: first write a test that reproduces the bug and confirm it fails. Then fix.
- FEATURE: write tests for the acceptance criteria alongside the implementation.
- Run the relevant tests after each meaningful change.

If the same test or error fails twice, STOP. Send `reviewer`:
- the error output
- both attempts and why each failed
- your current hypothesis

Follow its guidance before trying a third time.

## 4. Verify with screenshots
Screenshots are required. A task is not done without them.
Save them to `~/.claude/ship-evidence/<project>/<YYYY-MM-DD>-<short-slug>/` with descriptive names such as `01-before-double-total.png` or `02-after-single-coupon.png`. Keep them out of the repo.

- UI changes: run the app and capture the feature working, or the bug no longer happening. For bugs, also capture the broken state before the fix if you can still reproduce it (stash the fix or capture it before you start building).
  Use whatever is available, in this order: the `run` skill, a browser tool (Claude in Chrome, the built-in browser, or the T3 preview), then `npx playwright screenshot <url> <file>`.
- No UI (backend, CLI, library): screenshot the passing test run. Write the terminal output to an HTML page and capture it with Playwright, and keep the plain-text output too.
- Open each screenshot after you capture it and confirm it actually shows success: no error overlays, no blank pages, and the right state.

If you can't capture a screenshot (for example, the app needs credentials you don't have), say exactly why and stop instead of claiming it's verified.

## 5. Finish → Checkpoint C (reviewer)
Before saying you're done, ask `reviewer` to review the full change: tell it to run `git status` and `git diff` itself and check them against the task and the plan. Give it the screenshot folder path so it can check the evidence.
Fix every BLOCKING item, re-run the tests, and re-take any screenshots the fix affects.
Do not commit, push, or deploy.

## Report
- What changed (files and a one-line summary each)
- Test results
- Screenshots embedded inline as Markdown images using their absolute paths, each with a one-line caption of what it proves
- Each reviewer verdict, one line per checkpoint
- Anything left for me to decide
