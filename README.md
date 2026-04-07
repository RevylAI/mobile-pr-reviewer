# Mobile PR Reviewer

> Part of [Mobile DevTools](https://github.com/RevylAI/mobile-devtools) — open-source tools for mobile engineering teams.

AI-powered visual PR reviews for mobile apps. Two modes:

1. **Interactive** — Claude boots a cloud device, drives it with natural-language steps, and posts a link to the session recording
2. **E2E Test** — Claude creates a YAML test from the diff, runs it on Revyl's platform, and posts a shareable report link

> Developer pushes a button color change → Claude boots a phone in the cloud → drives through to the screen → posts a link to the full recording and step timeline in the PR. Automatically.

## How It Works

### Mode 1: Interactive Review (default)

```
PR opened
  → GitHub Actions builds the app
  → Uploads build to Revyl cloud (captures build_version_id for this PR)
  → Claude Code Action triggers:
      1. Reads the git diff against the PR's base branch
      2. Figures out what screens changed
      3. Starts a cloud device pinned to this PR's build
      4. Drives the device with natural language
         (revyl device instruction "Tap the Add to Cart button")
      5. Posts a PR comment with the session recording link
```

Triggered automatically on PR open. Re-trigger by commenting `/review` on the PR (repo collaborators only).

**Evidence is the session recording**, not screenshots. The recording includes the video of the device, every step Claude issued, and the result of each step — so reviewers get the full audit trail from a single link (`https://app.revyl.ai/sessions/<session_id>`).

### Mode 2: E2E Test Review

```
Comment "/test" on a PR
  → Claude Code Action triggers:
      1. Reads the git diff
      2. Creates a YAML E2E test definition targeting the changed flows
      3. Pushes the test to Revyl's cloud platform
      4. Runs the test against the latest uploaded build
      5. Generates a shareable report link
      6. Posts results with the report link as a PR comment
```

Triggered by commenting `/test` on a PR (requires a build from a prior PR push).

### Example PR Comments

**Interactive mode** — a single recording link as evidence:

> **Result:** ❌ Bug reproduced — Orchid Mantis → Gold Tortoise substitution confirmed.
>
> **Session recording:** [View full recording and step timeline](https://app.revyl.ai/sessions/sess_abc123)

The recording includes the video of the device, every step Claude issued (`instruction: Tap the Orchid Mantis card`, `validation: The cart contains Orchid Mantis at $62.00`, …), and the result of each step. One click, full audit trail.

**Test mode** — Claude creates a focused E2E test from the diff, runs it on a cloud device, and catches the cart substitution bug:

> **Status:** ❌ Failed (bug caught)  |  **Report:** [View full report](https://app.revyl.ai/tests/report?taskId=4028df46-5cce-410c-bbe2-e40a6d42657d)  |  **Test steps:** 5 blocks

Full interactive mode example: [`examples/sample-pr-comment.md`](examples/sample-pr-comment.md)

## Setup (5 minutes)

### 1. Fork this repo

Click **Fork** and clone it. The `sample-app/` directory contains [Bug Bazaar](sample-app/), a React Native e-commerce app you can use to test immediately.

### 2. Add 3 secrets

Go to **Settings → Secrets and variables → Actions** and add:

| Secret | Where to get it |
|--------|----------------|
| `ANTHROPIC_API_KEY` | [console.anthropic.com](https://console.anthropic.com/) |
| `REVYL_API_KEY` | [app.revyl.ai](https://app.revyl.ai) → Settings → API Keys |
| `REVYL_APP_ID` | Run `revyl app create --name "my-app" --platform android --json` |

### 3. Open a PR

Make any change to `sample-app/` and open a pull request.

- **Interactive review** runs automatically on PR open
- **E2E test review** runs when you comment `/test` on the PR

That's it. Three secrets, one workflow file, two review modes.

## Try It Now

The sample app has an **intentional bug** you can use to test:

```tsx
// sample-app/context/CartContext.tsx line 38-39
// Adding "Orchid Mantis" (id:3) silently adds "Gold Tortoise" (id:4) instead
```

Open a PR that "fixes" this bug — change `id === 3` back to the correct product.

- **Interactive:** Claude boots a device, adds Orchid Mantis to cart, and catches the substitution
- **Test:** Claude creates a test that validates cart contents after adding Orchid Mantis

## Customize for Your App

This repo is a template. To use it with your own mobile app:

### 1. Replace the sample app

Delete `sample-app/` and add your own app source (or just point the build step at your existing CI):

```yaml
# .github/workflows/review.yml — customize the build step
- name: Build app
  run: |
    # Gradle (Android)
    ./gradlew assembleDebug

    # Xcode (iOS)
    # xcodebuild -scheme MyApp -sdk iphonesimulator -configuration Debug

    # Flutter
    # flutter build apk --debug

    # React Native
    # npx react-native build-android --mode=debug
```

### 2. Update CLAUDE.md and CLAUDE-test.md

Tell Claude about your app's screens and navigation:

```markdown
## Your App

- **Login** — Email + password, then OTP verification
- **Dashboard** — Stats cards, recent activity feed
- **Settings** — Profile, notifications, theme toggle

Navigation: Bottom tabs (Dashboard, Activity, Settings). Login is shown when not authenticated.
```

The more context you give Claude about your app, the better it navigates (interactive) or designs tests (test mode).

### 3. Set your Revyl app ID

```bash
# Create your app on Revyl
revyl app create --name "MyApp" --platform android --json

# Add the returned ID as REVYL_APP_ID secret
```

### 4. (Optional) Use with existing CI

If you already have CI that builds your app, skip the build job and just download the artifact:

```yaml
review:
  # Remove the "needs: build" line
  steps:
    # Download from your existing build pipeline
    - uses: actions/download-artifact@v4
      with:
        name: my-app-build
        path: build/
    # ... rest of review job
```

## Architecture

```
.github/workflows/review.yml   # GitHub Actions: build → upload → Claude reviews
CLAUDE.md                       # Instructions for interactive mode (live device)
CLAUDE-test.md                  # Instructions for test mode (YAML test → report)
sample-app/                     # Bug Bazaar — demo React Native e-commerce app
examples/
└── sample-pr-comment.md        # What the PR comment looks like
```

The review bot is **three files**: `review.yml` (the trigger), `CLAUDE.md` (interactive instructions), and `CLAUDE-test.md` (test mode instructions). Everything else is the sample app.

## How the Revyl CLI Works

**Interactive mode** uses natural-language live steps. Each step is recorded to the session timeline so a single link in the PR comment is a full audit trail:

```bash
# Start a device pinned to the PR's build
revyl device start --platform android --app-id "$REVYL_APP_ID" \
                   --build-version-id "$REVYL_BUILD_VERSION_ID" --json

# Drive with high-level primitives — these show up as structured blocks in the recording
revyl device instruction "Tap the Add to Cart button" --json
revyl device validation  "The cart shows Orchid Mantis at \$62.00" --json

# Clean up
revyl device stop --all --json
```

**Test mode** uses structured YAML tests:

```bash
# Create and run an E2E test
revyl test create my-test --from-file test.yaml --platform android --no-open
revyl test run my-test --json --verbose
revyl test share my-test  # → shareable report link
```

Both approaches use Revyl's cloud devices — no local emulators or physical devices needed.

## Built With

- [Revyl CLI](https://github.com/RevylAI/revyl-cli) — Cloud device provisioning and AI-grounded interaction
- [Claude Code Action](https://github.com/anthropics/claude-code-action) — Run Claude Code in GitHub Actions
- [Expo](https://expo.dev) — React Native framework (sample app)

## License

MIT
