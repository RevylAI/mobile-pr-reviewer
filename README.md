# Mobile PR Reviewer

> Part of [Mobile DevTools](https://github.com/RevylAI/mobile-devtools) — open-source tools for mobile engineering teams.

AI-powered visual PR reviews for mobile apps. Claude boots a real cloud device for every PR, **drives it like a human user using the Revyl CLI** — `tap`, `swipe`, `screenshot`, look at the screen, decide what to do next — and posts a single PR comment whose evidence is a link to the full session recording.

![Mobile PR Reviewer demo — Claude catching the Orchid Mantis cart bug on a Pixel 7](examples/demo.gif)

> Developer pushes a button-color change → GitHub Actions builds the app → Claude boots a phone in the cloud → reactively taps through to the screen → posts a link to the recording in the PR. Cross-platform (iOS + Android in parallel) by default.

```
┌─────────────┐    ┌────────────┐    ┌──────────────┐    ┌──────────────────┐
│  PR opened  │ →  │  build job │ →  │ Claude review│ →  │ PR comment with  │
│  on GitHub  │    │ (per OS)   │    │ on Revyl     │    │ recording link/s │
└─────────────┘    └────────────┘    └──────────────┘    └──────────────────┘
```

---

## What it does

The whole repo is built around one idea: **Claude uses the Revyl CLI to drive a real cloud device, and analyzes what it sees from the screenshots itself**. No high-level test planner, no YAML, no DSL. Claude is the planner.

For every PR:

1. The build job builds your app and uploads it to Revyl, capturing a `build_version_id` so the device session is pinned to *this* PR's binary.
2. Two parallel review jobs (one Android, one iOS) start a Revyl cloud device, hand control to Claude, and let it loop:

   ```
   1. revyl device screenshot → Read the PNG
   2. Decide the next action from what's on screen
   3. revyl device tap --target "..."   (or swipe / type)
   4. revyl device screenshot → Read the PNG
   5. Loop
   ```

3. When Claude is satisfied (or sees a bug), it posts a PR comment whose only evidence is a link to the Revyl session recording. The recording has the video and every action Claude took.

That's the whole product. **Two CLAUDE markdown files, one workflow file, and the sample app.**

### Optional: trigger an existing Revyl test or workflow

If your team already has Revyl tests or workflows authored on the platform, the bot can trigger one against the PR build instead of (or in addition to) the reactive review. Add a `REVYL_TEST_NAME` or `REVYL_WORKFLOW_NAME` repo secret, then comment `/test` on a PR. Claude runs `revyl test run "$REVYL_TEST_NAME" --json --verbose` (or the workflow equivalent), parses `report_link` from the output, and posts that link to the PR. No test design, no YAML — just trigger-and-report. See `CLAUDE-test.md`.

---

## Real recordings — the Bug Bazaar demo

The included sample app (`sample-app/`) is **Bug Bazaar**, a React Native e-commerce app with an intentional bug: adding "Orchid Mantis" silently swaps in "Gold Tortoise" at the cart layer. Claude was given the diff for a PR that "fixes" this bug, with no hint about whether the build it was testing actually contained the fix. Both runs caught the bug autonomously by reactively tapping through the app.

| Platform | Recording | Device | Duration |
|---|---|---|---|
| **Android** | [session 76a6daae](https://app.revyl.ai/sessions/76a6daae-0107-459c-b416-c38a58e1f4ae) | Pixel 7 | 1:36 |
| **iOS** | [session 2960f466](https://app.revyl.ai/sessions/2960f466-a49a-4321-9597-9262276a787f) | iPhone 17 Pro Max | 1:34 |

The GIF above is the Android run, sped up 2× and converted to a small loop. Open either session link to watch the full video plus the raw step-by-step actions Claude took. A full sample PR comment is in [`examples/sample-pr-comment.md`](examples/sample-pr-comment.md).

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

**Optional** (only for the `/test` mode):

| Secret | What it does |
|---|---|
| `REVYL_TEST_NAME` | Name of an existing Revyl test to trigger when someone comments `/test` on a PR |
| `REVYL_WORKFLOW_NAME` | Name of an existing Revyl workflow to trigger on `/test` (set this OR `REVYL_TEST_NAME`, not both — workflows take precedence) |

Leave both unset if you don't want the `/test` mode.

### 3. Open a PR

Make any change to `sample-app/` (or your own app source) and open a pull request.

- The reactive review fires automatically on PR open across both platforms.
- Re-trigger with `/review` (collaborators only).
- If you've configured a Revyl test/workflow, comment `/test` to trigger it.

That's it. Five required secrets, two optional, one workflow file, two review modes (one default + one optional), one canonical demo.

---

## Try It Now

The sample app has an **intentional bug** in `sample-app/context/CartContext.tsx`:

```tsx
// Adding "Orchid Mantis" (id:3) silently adds "Gold Tortoise" (id:4) instead
const actualProduct = product.id === 3
  ? allProducts.find(p => p.id === 4)!
  : product;
```

Open a PR that "fixes" this by removing the substitution and using `product` directly. Claude will boot Android and iOS in parallel, navigate to Orchid Mantis on each, tap ADD TO CART, open the cart, and reproduce the swap bug — autonomously, with no hint about what the build contains.

---

## Customize for your app

The whole template is **one workflow file + two CLAUDE markdown files + one sample app**. Editing it is straightforward.

### Pick which platforms run

`.github/workflows/review.yml` has two jobs that use a `strategy.matrix.platform`. To skip a platform, edit the matrix:

```yaml
strategy:
  matrix:
    platform: [android, ios]   # ← drop one to skip a platform
```

### Disable the `/test` trigger mode entirely

Either delete the `test-review` job, or just leave both `REVYL_TEST_NAME` and `REVYL_WORKFLOW_NAME` unset — the job will skip itself silently.

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

Tell Claude about your app's screens. `CLAUDE.md` has a clearly-labelled "Demo hint — delete this section when you adapt this template" block at the bottom that you replace with your own context:

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
.github/workflows/review.yml    # GitHub Actions: build → review (matrix) → optional test-review
CLAUDE.md                        # Reactive review instructions (the main flow)
CLAUDE-test.md                   # OPTIONAL: trigger an existing Revyl test/workflow
sample-app/                      # Bug Bazaar — demo React Native e-commerce app
examples/
├── demo.gif                     # Inline animated demo for the README
└── sample-pr-comment.md         # Real PR comments from local harness runs
```

The review bot is **three files**: `review.yml` (the trigger + build), `CLAUDE.md` (reactive review), `CLAUDE-test.md` (optional trigger-existing-test). Everything else is the sample app.

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
   └─ review job (matrix: android, ios) — needs: build
         ├─ resolves PR base ref via gh api
         ├─ reads CLAUDE.md
         ├─ revyl device start --platform <p> --build-version-id <id>
         ├─ vision loop: screenshot → tap → screenshot → …
         ├─ revyl device stop --all
         └─ posts PR comment with https://app.revyl.ai/sessions/<id>
```

The `/test` comment trigger runs `test-review` instead, which doesn't touch `revyl device` at all — it just calls `revyl test run` or `revyl workflow run` and posts the report link.

### Security

- The `/review` and `/test` comment triggers are gated to `OWNER` / `MEMBER` / `COLLABORATOR` (`author_association` check). External commenters on a public fork cannot burn API credits.
- PR-open trigger runs on `pull_request` events from same-repo branches. Forks pulling against this template don't get the secrets — GitHub strips them by default for fork PRs, so the build job will fail loudly rather than leaking credentials.
- Newer commits / comments cancel in-flight runs via `concurrency:` so a long-running session doesn't block the next push.

---

## How the Revyl CLI works

The core primitives the workflow + `CLAUDE.md` use:

```bash
# Start a cloud device pinned to a specific build
revyl device start --platform <android|ios> \
                   --app-id "$REVYL_APP_ID" \
                   --build-version-id "$REVYL_BUILD_VERSION_ID" \
                   --json

# Reactive primitives — the entire toolset for the default review job
revyl device screenshot --out /tmp/screen.png
revyl device tap        --target "Add to Cart button" --json
revyl device swipe      --direction up --json
revyl device type       --target "Search field" --text "beetles" --json
revyl device go-home    --json
revyl device launch     --bundle-id <pkg> --json

# Always clean up
revyl device stop --all --json
```

For the **optional `/test` mode** (triggers an existing Revyl test/workflow):

```bash
# Run an existing test by name
revyl test run my-test --json --verbose
# → output JSON includes report_link directly, no separate share call needed

# Or run an existing workflow by name
revyl workflow run my-workflow --json
```

Both modes use Revyl's cloud devices — no local emulators or physical devices needed.

---

## Built With

- [Revyl CLI](https://github.com/RevylAI/revyl-cli) — Cloud device provisioning, AI-grounded interaction, session recordings
- [Claude Code Action](https://github.com/anthropics/claude-code-action) — Run Claude Code in GitHub Actions
- [Expo](https://expo.dev) — React Native framework (sample app)

## License

MIT
