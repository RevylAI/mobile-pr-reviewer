# Sample PR Comments

These are real PR comments mobile-pr-reviewer produced on local harness runs against the included Bug Bazaar sample app, on real Revyl cloud devices. The Orchid Mantis bug is intentional in `sample-app/context/CartContext.tsx`.

**The harness ran with no hint about the build state.** Claude read the diff, formed its own hypothesis ("the fix means the cart should contain Orchid Mantis at $62.00"), drove the device to validate that hypothesis, and reported what it found. In every cell of the matrix below, the validation **failed** because the build still had the bug — that's the autonomous bug catch.

The session recordings are the entire audit trail. No screenshots, no embedded images.

---

## Mode 1, structured (`CLAUDE.md`) — Android (canonical demo)

[**Open the recording →**](https://app.revyl.ai/sessions/67fa9104-3a87-4d38-ad9b-ed12d96cd5ee)

```markdown
## 📱 Mobile PR Review

**Changes detected:** This PR removes the bug in `CartContext.tsx` where adding Orchid Mantis (product ID 3) would silently swap it for Gold Tortoise (product ID 4) before adding to cart.

**Test plan:**
1. Navigate to Shop tab
2. Tap the Orchid Mantis product card
3. Verify product detail shows Orchid Mantis at $62.00
4. Tap ADD TO CART
5. Verify cart contains Orchid Mantis at $62.00 (not Gold Tortoise)

**Result:** ❌ **Validation failed**

**Expected:** Cart should contain Orchid Mantis at $62.00
**Observed:** Cart contains Gold Tortoise at $54.00

The bug described in the PR is still present in this build. Adding Orchid Mantis to the cart still results in Gold Tortoise being added instead. The fix in `CartContext.tsx` does not appear to be included in the tested build.

**Session recording:** [View full recording and step timeline](https://app.revyl.ai/sessions/67fa9104-3a87-4d38-ad9b-ed12d96cd5ee)

_Tested on Revyl Pixel 7 cloud device · Android · structured mode_
```

**Recording:** Pixel 7, 3:49, 7 timeline steps, **2/3 validations passed** (the cart-contents validation failed → bug caught). The recording shows the failed validation as a red entry in the step timeline — no human reviewer needs to watch the full video to see what went wrong.

**Step timeline (from the recording):**

```
1. instruction  Tap the Shop tab in the bottom navigation              ✓
2. validation   The shop screen shows a grid of product cards          ✓
3. instruction  Tap the Orchid Mantis product card                     ✓
4. validation   The product detail page shows 'Orchid Mantis' at $62   ✓
5. instruction  Tap the ADD TO CART button                             ✓
6. instruction  Tap the cart icon in the top right corner              ✓
7. validation   The cart contains Orchid Mantis at $62.00              ✗ ← bug caught
```

This is the canonical demo — both the README's animated GIF and the linked session above are this same Pixel 7 run.

---

## Mode 1, structured (`CLAUDE.md`) — iOS

[**Open the recording →**](https://app.revyl.ai/sessions/74c6401b-edbf-495d-ae2a-a297088c6241)

```markdown
## 📱 Mobile PR Review

**Changes detected:** Removes the product swap bug in `CartContext.tsx` where adding "Orchid Mantis" (ID 3) silently added "Gold Tortoise" (ID 4) to the cart instead.

**Test plan:**
1. Navigate to the Orchid Mantis product detail page
2. Verify product name shows "Orchid Mantis" at $62.00
3. Tap ADD TO CART
4. Open cart and verify contents

**Result:** ❌ **Bug still present in this build**

**Expected:** Cart contains "Orchid Mantis" at $62.00
**Observed:** Cart contains "Gold Tortoise" at $36.00

The PR's code change correctly removes the swap logic, but this build does not include the fix. The product swap bug is still active.

**Session recording:** [View full recording and step timeline](https://app.revyl.ai/sessions/74c6401b-edbf-495d-ae2a-a297088c6241)

_Tested on Revyl iPhone 17 Pro Max cloud device · iOS · structured mode_
```

**Recording:** iPhone 17 Pro Max, 3:58, 9 timeline steps, **1/4 validations passed** (cart-content assertions failed). Same shape as the Android run, same autonomous bug catch — the structured-mode validation against the post-fix expected state correctly fails when the bug is still in the build.

---

## Mode 1, reactive (`CLAUDE-reactive.md`) — Android

[**Open the recording →**](https://app.revyl.ai/sessions/76a6daae-0107-459c-b416-c38a58e1f4ae)

```markdown
## 📱 Mobile PR Review (reactive)

**Changes detected:** The PR removes the bug where adding Orchid Mantis (product ID 3) to the cart incorrectly swapped it for Gold Tortoise (product ID 4).

**Hypothesis:** Cart should contain Orchid Mantis at $62.00 after tapping ADD TO CART, not Gold Tortoise at $18.00.

**What I observed:** Starting from the Shop screen, I tapped the Orchid Mantis product card and confirmed the product detail page showed "Orchid Mantis" at $62.00. After tapping ADD TO CART, the cart modal opened showing **Gold Tortoise at $18.00** instead of the expected Orchid Mantis.

**Result:** ❌ The bug is still present in this build

**Expected:** Cart contains Orchid Mantis at $62.00
**Observed:** Cart contains Gold Tortoise at $18.00

**Session recording:** [View full recording](https://app.revyl.ai/sessions/76a6daae-0107-459c-b416-c38a58e1f4ae)

_Tested on Revyl Pixel 7 cloud device · Android · reactive mode_
```

**Recording:** Pixel 7, 1:36, **single "manual control" timeline entry** (this is the trade-off — reactive mode drives the device with raw `revyl device tap` and `screenshot`, which don't create per-action step entries on the recording side; the video is the whole audit trail). Faster than structured but the recording timeline is much less inspectable.

---

## Mode 1, reactive (`CLAUDE-reactive.md`) — iOS

[**Open the recording →**](https://app.revyl.ai/sessions/2960f466-a49a-4321-9597-9262276a787f)

```markdown
## 📱 Mobile PR Review (reactive)

**Changes detected:** This PR removes the bug in `CartContext.tsx` where adding Orchid Mantis (product ID 3) silently swapped in Gold Tortoise (product ID 4) before adding it to the cart.

**Hypothesis:** After this fix, tapping ADD TO CART on the Orchid Mantis product detail page should add Orchid Mantis at $62.00 to the cart, not Gold Tortoise at $18.00.

**What I observed:** I navigated to the Orchid Mantis product detail page (confirmed $62.00 price), tapped the ADD TO CART button, and the cart modal opened automatically. The cart contained Gold Tortoise at $18.00 instead of the expected Orchid Mantis at $62.00, confirming the bug is still present in this build.

**Result:** ❌ The fix is not present in this build - the product swap bug still occurs.

**Expected:** Cart should contain Orchid Mantis at $62.00
**Observed:** Cart contains Gold Tortoise at $18.00

**Session recording:** [View full recording](https://app.revyl.ai/sessions/2960f466-a49a-4321-9597-9262276a787f)

_Tested on Revyl iPhone 17 Pro Max cloud device · iOS · reactive mode_
```

**Recording:** iPhone 17 Pro Max, 1:34, single manual-control entry. Same trade-off as the Android reactive run.

---

## Reading the table

| | Structured | Reactive |
|---|---|---|
| **Android device session** | 3:49 | 1:36 |
| **iOS device session** | 3:58 | 1:34 |
| **Recording timeline** | ~7–9 typed `instruction`/`validation` entries with descriptions and statuses | Single `manual control` entry |
| **Bug catch is visible in** | the timeline itself (failed validation = red row) | the PR comment text + the video |
| **CLI primitives used** | `revyl device instruction "..."`, `revyl device validation "..."` | `revyl device tap --target "..."`, `revyl device screenshot`, `revyl device swipe` |
| **Best for** | Reviewers who want to skim a step list before opening the video | Apps where the high-level grounder doesn't resolve targets reliably, or where you want maximum control |

Both modes pin the device to the PR's exact build via `--build-version-id` and clean up via `revyl device stop --all` even on failure. The default workflow runs **reactive** across both Android and iOS in parallel on every PR open. Customers who prefer the structured timeline can swap the default with a one-line edit to the workflow file.
