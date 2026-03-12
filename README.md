# Mobile PR Reviewer

AI-powered visual PR reviews for mobile apps. When a developer opens a pull request, Claude Code analyzes the diff, boots a cloud device with the new build, navigates to the changed screen, validates the change works, and posts screenshots directly in the PR.

> Developer pushes a button color change → Claude boots a phone in the cloud → taps through to the screen → screenshots the result → posts it in the PR. Automatically.

## How It Works

```
PR opened
  → GitHub Actions builds the app
  → Uploads build to Revyl cloud
  → Claude Code Action triggers:
      1. Reads the git diff
      2. Figures out what screens changed
      3. Starts a cloud device with the new build
      4. Navigates to the changed screen using natural language
         (e.g. revyl device tap --target "Add to Cart button")
      5. Takes screenshots as evidence
      6. Posts results as a PR comment
```

### Example PR Comment

When a developer changes the "Add to Cart" button styling, Claude posts:

> **📱 Mobile PR Review**
>
> **Changes detected:** Updated the "Add to Cart" button — changed from outline to filled blue with cart icon.
>
> **✅ Product detail page renders correctly**
> | Before | After |
> |--------|-------|
> | ![before](screenshot) | ![after](screenshot) |
>
> **✅ Add to Cart interaction works** — cart badge updated to "1"
>
> **Result:** ✅ All changes validated

Full example: [`examples/sample-pr-comment.md`](examples/sample-pr-comment.md)

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

Make any change to `sample-app/` and open a pull request. Claude will:
1. Build the app
2. Upload it to Revyl
3. Start a cloud device
4. Validate your change with screenshots
5. Post results as a PR comment

That's it. Three secrets, one workflow file.

## Try It Now

The sample app has an **intentional bug** you can use to test:

```tsx
// sample-app/context/CartContext.tsx line 38-39
// Adding "Orchid Mantis" (id:3) silently adds "Gold Tortoise" (id:4) instead
```

Open a PR that "fixes" this bug — change `id === 3` back to the correct product. Claude will boot a device, add the Orchid Mantis to cart, and verify the fix actually works.

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

### 2. Update CLAUDE.md

Tell Claude about your app's screens and navigation:

```markdown
## Your App

- **Login** — Email + password, then OTP verification
- **Dashboard** — Stats cards, recent activity feed
- **Settings** — Profile, notifications, theme toggle

Navigation: Bottom tabs (Dashboard, Activity, Settings). Login is shown when not authenticated.
```

The more context you give Claude about your app, the better it navigates.

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
CLAUDE.md                       # Instructions for Claude (the brain of the bot)
sample-app/                     # Bug Bazaar — demo React Native e-commerce app
examples/
└── sample-pr-comment.md        # What the PR comment looks like
```

The entire review bot is **two files**: `review.yml` (the trigger) and `CLAUDE.md` (the instructions). Everything else is the sample app.

## How the Revyl CLI Works

The magic is in [Revyl's](https://revyl.ai) `--target` flag — it uses AI to resolve natural language descriptions to screen coordinates:

```bash
# These just work — no accessibility IDs, no XPaths, no element inspectors
revyl device tap --target "Add to Cart button"
revyl device type --target "Search field" --text "beetles"
revyl device tap --target "Checkout button"
revyl device screenshot --out evidence.png
```

This means Claude can navigate **any app** without knowing the UI hierarchy. It reads the screen, decides what to tap, and uses natural language to do it.

## Built With

- [Revyl CLI](https://revyl.ai) — Cloud device provisioning and AI-grounded interaction
- [Claude Code Action](https://github.com/anthropics/claude-code-action) — Run Claude Code in GitHub Actions
- [Expo](https://expo.dev) — React Native framework (sample app)

## License

MIT
