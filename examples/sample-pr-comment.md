# Sample PR Comments

These are real PR comments mobile-pr-reviewer produced on local harness runs against the included Bug Bazaar sample app, on Revyl cloud simulators. The Orchid Mantis bug is intentional in `sample-app/context/CartContext.tsx`.

**The harness ran with no hint about the build state.** Claude read the diff, formed its own hypothesis ("the fix means the cart should contain Orchid Mantis at $62.00 after tapping ADD TO CART"), drove the device through the reactive loop in `CLAUDE.md`, and reported what it observed. Both runs **caught the bug autonomously** — the build still has the swap, the cart shows Gold Tortoise instead of Orchid Mantis, Claude reported ❌.

The session recordings are the entire audit trail. No screenshots in the comment, no embedded images.

---

## Android — Pixel 7 (canonical demo, also the GIF in the README)

[**Open the recording →**](https://app.revyl.ai/sessions/76a6daae-0107-459c-b416-c38a58e1f4ae)

```markdown
## 📱 Mobile PR Review

**Changes detected:** The PR removes the bug where adding Orchid Mantis (product ID 3) to the cart incorrectly swapped it for Gold Tortoise (product ID 4).

**Hypothesis:** Cart should contain Orchid Mantis at $62.00 after tapping ADD TO CART, not Gold Tortoise at $18.00.

**What I observed:** Starting from the Shop screen, I tapped the Orchid Mantis product card and confirmed the product detail page showed "Orchid Mantis" at $62.00. After tapping ADD TO CART, the cart modal opened showing **Gold Tortoise at $18.00** instead of the expected Orchid Mantis.

**Result:** ❌ The bug is still present in this build

**Expected:** Cart contains Orchid Mantis at $62.00
**Observed:** Cart contains Gold Tortoise at $18.00

**Session recording:** [View full recording](https://app.revyl.ai/sessions/76a6daae-0107-459c-b416-c38a58e1f4ae)

_Tested on Revyl Pixel 7 cloud simulator · Android_
```

**Recording:** Pixel 7, 1:36 device session. The animated GIF at the top of the README is a 2× speedup of this exact session. Open the link to watch the full video and see every action Claude took.

---

## iOS — iPhone 17 Pro Max

[**Open the recording →**](https://app.revyl.ai/sessions/2960f466-a49a-4321-9597-9262276a787f)

```markdown
## 📱 Mobile PR Review

**Changes detected:** This PR removes the bug in `CartContext.tsx` where adding Orchid Mantis (product ID 3) silently swapped in Gold Tortoise (product ID 4) before adding it to the cart.

**Hypothesis:** After this fix, tapping ADD TO CART on the Orchid Mantis product detail page should add Orchid Mantis at $62.00 to the cart, not Gold Tortoise at $18.00.

**What I observed:** I navigated to the Orchid Mantis product detail page (confirmed $62.00 price), tapped the ADD TO CART button, and the cart modal opened automatically. The cart contained Gold Tortoise at $18.00 instead of the expected Orchid Mantis at $62.00, confirming the bug is still present in this build.

**Result:** ❌ The fix is not present in this build - the product swap bug still occurs.

**Expected:** Cart should contain Orchid Mantis at $62.00
**Observed:** Cart contains Gold Tortoise at $18.00

**Session recording:** [View full recording](https://app.revyl.ai/sessions/2960f466-a49a-4321-9597-9262276a787f)

_Tested on Revyl iPhone 17 Pro Max cloud simulator · iOS_
```

**Recording:** iPhone 17 Pro Max, 1:34 device session. Same shape as the Android run, same autonomous bug catch — but on iOS this time, in parallel during the same matrix workflow run.

---

## What both runs have in common

Both Claude sessions used **only** raw `revyl device` primitives — `tap`, `swipe`, `screenshot` — inside a tight see → decide → act loop. No `revyl device instruction`, no `revyl device validation`, no test plan written ahead of time, no YAML.

The recording on Revyl shows the video of the device plus every command Claude ran. That's the entire audit trail in one URL.

---

## Optional: triggering an existing Revyl test or workflow

If your team already has Revyl tests or workflows authored on the platform, the bot can trigger one against the PR build instead of (or in addition to) the reactive review above. See `CLAUDE-test.md` and the `test-review` job in `.github/workflows/review.yml`. The PR comment in that mode is even simpler — just the report link from `revyl test run --json`:

```markdown
## 📱 Mobile PR Review — Triggered Revyl test

**Triggered:** `cart-add-to-cart-flow`

**Status:** ❌ Failed

**Report:** [View full report and recording](https://app.revyl.ai/tests/report?taskId=...)
```

That mode is opt-in — set `REVYL_TEST_NAME` or `REVYL_WORKFLOW_NAME` as a repo secret to enable it.
