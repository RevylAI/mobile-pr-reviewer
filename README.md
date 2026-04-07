# Mobile PR Reviewer

> Part of [Mobile DevTools](https://github.com/RevylAI/mobile-devtools) — open-source tools for mobile engineering teams.

AI-powered visual PR reviews for mobile apps. Claude boots a real cloud device for every PR, drives it through the screens your PR changed, and posts a single PR comment whose evidence is a link to the full session recording.

> Developer pushes a button-color change → GitHub Actions builds the app → Claude boots a phone in the cloud → drives through to the screen → posts a link to the recording in the PR. Cross-platform (iOS + Android in parallel) by default.

```
┌─────────────┐    ┌────────────┐    ┌──────────────┐    ┌──────────────────┐
│  PR opened  │ →  │  build job │ →  │ Claude review│ →  │ PR comment with  │
│  on GitHub  │    │ (per OS)   │    │ on Revyl     │    │ recording link/s │
└─────────────┘    └────────────┘    └──────────────┘    └──────────────────┘
```

---

## What it does

Mobile PR Reviewer ships **three review modes**. You can run any combination — they're three independent jobs in the workflow file. Each job is opt-in or opt-out via straight YAML editing, no DSL.

| Mode | Trigger | What Claude does | When to use it |
|---|---|---|---|
| **Reactive** (mode 1, default) | Auto on PR open · re-run with `/review` | Drives the device with a vision loop: `screenshot → decide → tap/swipe → screenshot → …`. Pure raw device commands. | Default for most apps. Maximum control, faithful to what was actually clicked. |
| **Structured** (mode 1, opt-in) | `/review-structured` comment | Drives the device with high-level `revyl device instruction` / `validation` primitives so the recording timeline reads as named natural-language steps. | When the recording's step timeline matters more than raw control. Slightly higher per-step latency but cleaner for human reviewers to skim. |
| **E2E Test** (mode 2) | `/test` comment | Designs a focused YAML E2E test from the diff, creates it on Revyl, runs it on the latest uploaded build, posts the report link. | When you want a structured test artifact you can re-run later. |

All three modes produce the same shape of PR comment: short summary → **single link** to the Revyl session/report. No embedded screenshots, no comparison tables — the recording is the entire audit trail.

### Cross-platform by default

The default workflow runs both interactive jobs as a `[android, ios]` matrix, in parallel. So a single PR open triggers two cloud-device runs (one Android, one iOS), each posting its own PR comment with its own recording. Drop a platform by editing one line.

---

## Real recordings — the canonical Bug Bazaar demo

The included sample app (`sample-app/`) is **Bug Bazaar**, a React Native e-commerce app with an intentional bug: adding "Orchid Mantis" silently swaps in "Gold Tortoise" at the cart layer. We exercised every cell of the matrix against a "fix" PR diff using a local harness — Claude reproduced the bug in all four cells and posted PR comments matching the new template.

| | Structured (`CLAUDE.md`) | Reactive (`CLAUDE-reactive.md`) |
|---|---|---|
| **Android** | [session b9ef064b](https://app.revyl.ai/sessions/b9ef064b-949c-40c7-b3de-6e1a4d397963) — 4.6 min, $0.68, 24 tool calls | [session 93c81ba1](https://app.revyl.ai/sessions/93c81ba1-dbad-409f-b14f-67447ea8bd99) — 4.3 min, $0.45, 30 tool calls |
| **iOS** | [session 7e5fb852](https://app.revyl.ai/sessions/7e5fb852-ca67-4111-9aa8-d7e9fc5815ec) — 7.2 min, $0.28, 20 tool calls | [session 8ad07bc2](https://app.revyl.ai/sessions/8ad07bc2-0aba-4026-a14d-15b80e355977) — 4.6 min, $0.49, 32 tool calls |

Each link opens the device-session report: video recording, every step Claude issued, and the result of each step. Open one and you've seen the entire PR review without reading a line of text.

> The structured iOS run survived a mid-session expiry (called `revyl device stop`, restarted via `revyl device start`, continued the test) without any human intervention. That's the durability you want from a PR review bot.

A full sample PR comment is in [`examples/sample-pr-comment.md`](examples/sample-pr-comment.md).

---

## Setup (5 minutes)

### 1. Fork this repo

Click **Fork** and clone it. The `sample-app/` directory contains Bug Bazaar — it builds out of the box and has a real bug you can demo against.

### 2. Add 5 secrets

Go to **Settings → Secrets and variables → Actions** and add:

| Secret | Where to get it | Required for |
|---|---|---|
| `ANTHROPIC_API_KEY` | [console.anthropic.com](https://console.anthropic.com/) | All modes |
| `REVYL_API_KEY` | [app.revyl.ai](https://app.revyl.ai) → Settings → API Keys | All modes |
| `REVYL_APP_ID_ANDROID` | `revyl app create --name "myapp-android" --platform android --json` | Android matrix cells |
| `REVYL_APP_ID_IOS` | `revyl app create --name "myapp-ios" --platform ios --json` | iOS matrix cells |
| `EXPO_TOKEN` | [expo.dev](https://expo.dev) → Personal Access Tokens | The bundled sample app's Expo build only — not needed if you replace `sample-app/` with your own build step |

If you only support one platform, set only that platform's `REVYL_APP_ID_*` secret and drop the other from the workflow matrix (one line edit, see "Customize" below).

### 3. Open a PR

Make any change to `sample-app/` (or your own app source) and open a pull request.

- The default reactive review fires automatically on PR open across both platforms.
- Re-trigger with `/review` (collaborators only).
- Try the structured mode with `/review-structured`.
- Try the E2E test mode with `/test` (after at least one prior PR push has uploaded a build).

That's it. Five secrets, one workflow file, three review modes.

---

## Try It Now

The sample app has an **intentional bug** in `sample-app/context/CartContext.tsx`:

```tsx
// Adding "Orchid Mantis" (id:3) silently adds "Gold Tortoise" (id:4) instead
const actualProduct = product.id === 3
  ? allProducts.find(p => p.id === 4)!
  : product;
```

Open a PR that "fixes" this by removing the substitution and using `product` directly.

- **Reactive** (default): Claude boots both Android and iOS, navigates to Orchid Mantis, taps ADD TO CART, opens the cart, and reproduces the swap bug on whichever build still has it.
- **Structured** (`/review-structured`): same flow but the recording timeline reads as named steps.
- **Test** (`/test`): Claude creates a YAML test that asserts the cart contents.

---

## Customize for your app

The whole template is **one workflow file + three CLAUDE markdown files + one sample app**. Editing it is straightforward:

### Pick which modes / platforms run

`.github/workflows/review.yml` has three review jobs (`review-reactive`, `review-structured`, `test-review`) plus one `build` job. Each has a clearly-marked `strategy.matrix.platform` you can edit:

```yaml
strategy:
  matrix:
    platform: [android, ios]   # ← drop one to skip a platform
```

To **disable a mode entirely**, delete its job. To **flip the default mode-1 flavor** to structured, swap the `if:` blocks between `review-reactive` and `review-structured` so the structured one fires on `pull_request`.

### Replace the sample app

Delete `sample-app/` and add your own app source, or point the build step at your existing CI artifact. The build step in `review.yml` is the only thing that needs to know how to produce a binary:

```yaml
- name: Build app
  run: |
    if [ "${{ matrix.platform }}" = "android" ]; then
      ./gradlew assembleDebug
    else
      xcodebuild -scheme MyApp -sdk iphonesimulator -configuration Debug
    fi
```

If you already have CI that builds your app, you can drop the build job entirely and download the artifact in each review job:

```yaml
- uses: actions/download-artifact@v4
  with:
    name: my-app-${{ matrix.platform }}-build
```

### Update the CLAUDE files

Tell Claude about your app's screens. Each file has a clearly-labelled "Demo hint — delete this section when you adapt this template" block at the bottom that you replace with your own context:

```markdown
## Your App

- **Login** — Email + password, then OTP verification
- **Dashboard** — Stats cards, recent activity feed
- **Settings** — Profile, notifications, theme toggle

Navigation: Bottom tabs (Dashboard, Activity, Settings). Login is shown when not authenticated.
```

The more domain context you give Claude, the more confidently it navigates and the cleaner the recordings.

### Switch to a different model

Each Claude job uses `claude_args: --model claude-sonnet-4-5`. Swap to `claude-opus-4-6` for harder reviews (slower, more expensive, smarter), or `claude-haiku-4-5` for cheaper smoke tests.

---

## Architecture

```
.github/workflows/review.yml    # GitHub Actions: build → 3 independent review jobs
CLAUDE.md                        # Mode 1 STRUCTURED instructions (opt-in)
CLAUDE-reactive.md               # Mode 1 REACTIVE instructions (default)
CLAUDE-test.md                   # Mode 2 instructions (E2E YAML test)
sample-app/                      # Bug Bazaar — demo React Native e-commerce app
examples/
└── sample-pr-comment.md         # Real PR comment from a local harness run
```

The review bot is **four files**: `review.yml` (the trigger + build), `CLAUDE.md`, `CLAUDE-reactive.md`, and `CLAUDE-test.md`. Everything else is the sample app.

### How a PR open flows through the workflow

```
PR opened
   │
   ├─ build job (matrix: android, ios)
   │     ├─ npm ci sample-app
   │     ├─ eas-cli build --local --platform <android|ios>
   │     ├─ revyl build upload --json → captures build_version_id
   │     └─ exports per-platform build_version_id as job output
   │
   └─ review-reactive job (matrix: android, ios) — needs: build
         ├─ resolves PR base ref via gh api
         ├─ reads CLAUDE-reactive.md
         ├─ revyl device start --platform <p> --build-version-id <id>
         ├─ vision loop: screenshot → tap → screenshot → …
         ├─ revyl device stop --all
         └─ posts PR comment with https://app.revyl.ai/sessions/<id>
```

Comment-triggered flows (`/review`, `/review-structured`, `/test`) skip the build job entirely and reuse the latest uploaded build for that app.

### Security

- The `/review`, `/review-structured`, and `/test` comment triggers are gated to `OWNER` / `MEMBER` / `COLLABORATOR` (`author_association` check). External commenters on a public fork cannot burn API credits.
- PR-open trigger runs on `pull_request` events from same-repo branches. Forks pulling against this template don't get the secrets — GitHub strips them by default for fork PRs, so the build job will fail loudly rather than leaking credentials.
- Newer commits / comments cancel in-flight runs via `concurrency:` so a long-running session doesn't block the next push.

---

## How the Revyl CLI works

The core primitives the workflow / CLAUDE files use:

```bash
# Start a cloud device pinned to a specific build
revyl device start --platform <android|ios> \
                   --app-id "$REVYL_APP_ID" \
                   --build-version-id "$REVYL_BUILD_VERSION_ID" \
                   --json

# REACTIVE primitives (used by CLAUDE-reactive.md)
revyl device screenshot --out /tmp/screen.png
revyl device tap        --target "Add to Cart button" --json
revyl device swipe      --direction up --json
revyl device type       --target "Search field" --text "beetles" --json

# STRUCTURED primitives (used by CLAUDE.md)
revyl device instruction "Tap the Add to Cart button" --json
revyl device validation  "The cart contains Orchid Mantis at \$62.00" --json
revyl device extract     "the order total" --json

# Always clean up
revyl device stop --all --json
```

For **mode 2** (E2E test):

```bash
revyl test create my-test --from-file test.yaml --platform android --app "$REVYL_APP_ID" --no-open --force --json
revyl test run    my-test --json --verbose   # → output includes report_link directly
```

Both modes use Revyl's cloud devices — no local emulators or physical devices needed.

---

## Built With

- [Revyl CLI](https://github.com/RevylAI/revyl-cli) — Cloud device provisioning, AI-grounded interaction, session recordings
- [Claude Code Action](https://github.com/anthropics/claude-code-action) — Run Claude Code in GitHub Actions
- [Expo](https://expo.dev) — React Native framework (sample app)

## License

MIT
