# Mobile PR Review Agent — Trigger Existing Test or Workflow

This is the **secondary** mode. The primary mode is in `CLAUDE.md` — Claude drives the device reactively for every PR. This file is what Claude reads when you've already authored Revyl tests or workflows on your account and you want the bot to **trigger one of them** against the PR's build instead of (or in addition to) doing a free-form reactive review.

You don't write a test here. You don't design YAML. You take an existing test or workflow name, run it via the Revyl CLI, parse the report URL out of the JSON output, and post that to the PR.

## Your tools

- **`git`** / **`gh`** — read PR metadata (you don't need a deep diff for this mode)
- **`revyl`** — call exactly one of:
  - `revyl test run <name> --json --verbose` — run a Revyl test by name
  - `revyl workflow run <name> --json` — run a Revyl workflow by name

That's the whole tool surface. There is no device session here, no `revyl device start` — Revyl's test runner provisions its own simulator.

## Environment

The workflow sets these for you before you run:

- `REVYL_API_KEY` — Revyl auth
- `REVYL_APP_ID` — the Revyl app the test/workflow targets
- `REVYL_TEST_NAME` *(optional)* — the name of the Revyl test to run on `/test`
- `REVYL_WORKFLOW_NAME` *(optional)* — the name of the Revyl workflow to run on `/test`
- `PR_BASE_REF` — the PR's base branch (you almost certainly don't need it for this mode)

**Exactly one of `REVYL_TEST_NAME` or `REVYL_WORKFLOW_NAME` should be set.** If neither is set, the workflow will skip this job before Claude runs. If both are set, prefer `REVYL_WORKFLOW_NAME` (workflows compose tests, so they're usually the broader signal).

## Workflow

### 1. Decide which to run

```bash
if [ -n "$REVYL_WORKFLOW_NAME" ]; then
  mode=workflow
  name="$REVYL_WORKFLOW_NAME"
elif [ -n "$REVYL_TEST_NAME" ]; then
  mode=test
  name="$REVYL_TEST_NAME"
else
  echo "Neither REVYL_TEST_NAME nor REVYL_WORKFLOW_NAME is set. Nothing to do."
  exit 0
fi
echo "Running $mode: $name"
```

### 2. Run it

```bash
if [ "$mode" = "test" ]; then
  result=$(revyl test run "$name" --json --verbose)
else
  result=$(revyl workflow run "$name" --json)
fi
echo "$result"
```

`revyl test run` blocks until the test completes. `revyl workflow run` likewise. Both emit a JSON object that includes a shareable `report_link` (you don't need a separate `revyl test share` / `workflow share` call).

### 3. Parse the result

```bash
report_link=$(echo "$result" | jq -r '.report_link // empty')
status=$(echo "$result"      | jq -r '.status      // empty')
error_message=$(echo "$result" | jq -r '.error_message // empty')

if [ -z "$report_link" ]; then
  echo "ERROR: revyl did not return a report_link. Output was:"
  echo "$result"
  exit 1
fi
```

### 4. Post the PR comment

Lead with the report link. Keep it short — the report has all the detail.

```markdown
## 📱 Mobile PR Review — Triggered Revyl <test|workflow>

**Triggered:** `<name>` ([Revyl <test|workflow>](https://app.revyl.ai/<tests|workflows>))

**Status:** ✅ Passed  (or)  ❌ Failed

**Report:** [View full report and recording](<report_link>)

<If failed: one short paragraph with `error_message` or the failing-step summary from the report.>
```

That's it. No screenshots, no comparison tables — the Revyl report has the full step-by-step breakdown with video.

## Critical rules

- **Do not author a test here.** If you need to design new test logic, that's a job for a human (or the primary reactive mode in `CLAUDE.md`). This mode only runs tests/workflows that already exist on the customer's Revyl account.
- **Always pass `--json`** so output is parseable.
- **`revyl test run` blocks until the test completes.** Don't background it, don't poll separately.
- **Use `report_link` straight from the run output.** No `revyl test share` / `revyl workflow share` round trip.
- **If neither env var is set, exit 0 silently.** The workflow's job-level `if:` should already prevent this case from happening, but be defensive.
- **If the Revyl call errors out, post a short comment explaining what failed** (network error, unknown test name, etc.) instead of crashing the whole job.

## Why this mode exists

The primary mode in `CLAUDE.md` is Claude doing free-form reactive review on every PR. That's the canonical demo. **This mode is for customers who already have a curated suite of Revyl tests or workflows** and want their PR bot to also rerun those against new builds. You can run both:

- Default: every PR open triggers the reactive review (`CLAUDE.md`) across both platforms
- Optional: a `/test` comment additionally triggers your existing Revyl test/workflow (`CLAUDE-test.md`)

Or run only one — delete whichever job you don't need from `.github/workflows/review.yml`.
