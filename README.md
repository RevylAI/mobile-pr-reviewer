# Mobile PR Reviewer

> Part of [Mobile DevTools](https://github.com/RevylAI/mobile-devtools) — open-source tools for mobile engineering teams.

Claude reviews mobile PRs by booting a cloud simulator, tapping through the app, and posting a link to the recording.

![Mobile PR Reviewer demo](examples/demo.gif)

That's a real PR review. Claude read the diff, started a Pixel 7 simulator on Revyl, navigated to the changed screen, hit a bug, and posted a comment with the recording link. ~90 seconds. No test code, no YAML, no maintenance.

Two recordings from the bundled Bug Bazaar sample app — the same flow, different platforms in parallel:

- **Android** — [session 76a6daae](https://app.revyl.ai/sessions/76a6daae-0107-459c-b416-c38a58e1f4ae) · Pixel 7 · 1:36
- **iOS** — [session 2960f466](https://app.revyl.ai/sessions/2960f466-a49a-4321-9597-9262276a787f) · iPhone 17 Pro Max · 1:34

A full sample PR comment is in [`examples/sample-pr-comment.md`](examples/sample-pr-comment.md).

## Setup

1. Fork this repo. The bundled `sample-app/` is Bug Bazaar — it builds out of the box and has a real bug to demo against.
2. Add 5 secrets in **Settings → Secrets and variables → Actions**:

   | Secret | Where to get it |
   |---|---|
   | `ANTHROPIC_API_KEY` | [console.anthropic.com](https://console.anthropic.com/) |
   | `REVYL_API_KEY` | [app.revyl.ai](https://app.revyl.ai/) → Settings → API Keys |
   | `REVYL_APP_ID_ANDROID` | `revyl app create --name "myapp-android" --platform android --json` |
   | `REVYL_APP_ID_IOS` | `revyl app create --name "myapp-ios" --platform ios --json` |
   | `EXPO_TOKEN` | Only if your build uses Expo (the sample app does) |

3. Open a PR. The bot reviews on PR open and on `/review` comments.

If you only support one platform, set just that platform's `REVYL_APP_ID_*` and drop the other from `strategy.matrix.platform` in `.github/workflows/review.yml`.

## How it works

For each PR, two parallel jobs (Android + iOS) start a Revyl cloud simulator, hand control to Claude, and let it loop:

```
1. revyl device screenshot     → Read the PNG
2. Decide the next action from what's on screen
3. revyl device tap --target "..."   (or swipe / type)
4. Repeat
```

When Claude is done — or sees a bug — it posts a PR comment with the recording link. The recording has the video and every action Claude took. Claude is the planner; there's no test plan written ahead of time, no DSL, no YAML.

Three files the bot reads: [`CLAUDE.md`](CLAUDE.md) (the reactive loop), [`CLAUDE-test.md`](CLAUDE-test.md) (optional `/test` mode), [`.github/workflows/review.yml`](.github/workflows/review.yml) (the trigger). That's the whole product.

## Optional: trigger an existing Revyl test

If your team already has Revyl tests or workflows, the bot can trigger one against the PR build instead of doing a free-form review. Add one of these as a repo secret:

- `REVYL_TEST_NAME` — name of an existing Revyl test
- `REVYL_WORKFLOW_NAME` — name of an existing Revyl workflow

Then comment `/test` on a PR. Claude runs `revyl test run "$REVYL_TEST_NAME" --json --verbose`, parses `report_link`, posts it. No test design, no YAML — trigger and report. Leave both secrets unset to disable. See [`CLAUDE-test.md`](CLAUDE-test.md).

## Customize for your app

Replace `sample-app/` with your source, or point the `Build app` step in `.github/workflows/review.yml` at your existing CI artifact:

```yaml
- name: Build app
  run: |
    if [ "${{ matrix.platform }}" = "android" ]; then
      ./gradlew assembleDebug
    else
      xcodebuild -scheme MyApp -sdk iphonesimulator -configuration Debug
    fi
```

Then edit the bottom of `CLAUDE.md` (the "Demo hint" section) to describe your screens — bottom tabs, top-level routes, login flow, anything Claude needs to know to navigate confidently.

To swap models, edit `claude_args: --model claude-sonnet-4-5` in the workflow. `claude-opus-4-6` for harder reviews, `claude-haiku-4-5` for cheaper smoke tests.

## Security

- `/review` and `/test` comment triggers are gated to `OWNER` / `MEMBER` / `COLLABORATOR` via `author_association`. External commenters can't burn API credits.
- Forks pulling against this template don't get the secrets — GitHub strips them by default for fork PRs. The build job will fail loudly rather than leak.
- `concurrency:` cancels in-flight runs when newer commits land.

## License

MIT
