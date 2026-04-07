# Sample PR Comments

These are real PR comments mobile-pr-reviewer produced on local harness runs against the included Bug Bazaar sample app, on real Revyl cloud devices. The Orchid Mantis bug is intentional in `sample-app/context/CartContext.tsx` — Claude reproduced it from a one-line "fix" PR diff in every cell of the matrix.

The session recordings are the entire audit trail. No screenshots, no embedded images.

---

## Mode 1, structured (`CLAUDE.md`) — Android

[**Open the recording →**](https://app.revyl.ai/sessions/b9ef064b-949c-40c7-b3de-6e1a4d397963)

```markdown
## 📱 Mobile PR Review

**Changes detected:** Fixes cart bug where adding Orchid Mantis (product ID 3) incorrectly adds Gold Tortoise (product ID 4) instead. Removes the swap logic in `CartContext.tsx`.

**Test plan:**
1. Navigate to Orchid Mantis product detail page
2. Verify product shows Orchid Mantis at $62.00
3. Tap ADD TO CART button
4. Open cart and verify contents

**Result:** ❌ **Bug reproduced**

**Expected:** Cart should contain Orchid Mantis at $62.00  
**Observed:** Cart contains Gold Tortoise at $18.00

This confirms the bug described in the PR is present in the current build and validates the need for this fix.

**Session recording:** [View full recording and step timeline](https://app.revyl.ai/sessions/b9ef064b-949c-40c7-b3de-6e1a4d397963)

_Tested on Revyl cloud device · build `45d1a2fc`_
```

**Run stats:** 24 tool calls / 4.6 min / $0.68 / Sonnet 4.5  
**Revyl device commands:** `instruction` ×4, `validation` ×2, `screenshot` ×3 (Claude's own verification, never embedded), `start`/`stop` ×2/×1, **`tap` ×0**

The recording's step timeline reads as named natural-language blocks, exactly what reviewers want when they're skimming.

---

## Mode 1, reactive (`CLAUDE-reactive.md`) — Android

[**Open the recording →**](https://app.revyl.ai/sessions/93c81ba1-dbad-409f-b14f-67447ea8bd99)

```markdown
## 📱 Mobile PR Review (reactive)

**Changes detected:** This PR removes the product ID swap logic in `CartContext.tsx` that was causing Orchid Mantis (product ID 3) to be replaced with Gold Tortoise (product ID 4) when adding to cart.

**Hypothesis:** After the fix, adding Orchid Mantis to the cart should keep Orchid Mantis at $62.00, not swap it for Gold Tortoise at $18.00.

**What I observed:** I navigated to the Orchid Mantis product detail page, which correctly displayed "Orchid Mantis" with a price of $62.00. After tapping the ADD TO CART button, the cart modal opened showing **Gold Tortoise at $18.00** instead of the expected Orchid Mantis at $62.00. The bug is present in this build.

**Result:** ❌ **Bug reproduced** — Expected Orchid Mantis ($62.00) in cart, but found Gold Tortoise ($18.00) instead. This confirms the bug exists in the pre-fix build and validates that the PR addresses a real issue.

**Session recording:** [View full recording](https://app.revyl.ai/sessions/93c81ba1-dbad-409f-b14f-67447ea8bd99)

_Tested on Revyl `android` cloud device · build `45d1a2fc` · reactive mode_
```

**Run stats:** 30 tool calls / 4.3 min / $0.45 / Sonnet 4.5  
**Revyl device commands:** `screenshot` ×8, `tap` ×7, `start`/`stop` ×2/×1, **`instruction`/`validation` ×0**

Pure vision loop. Lower per-step cost, faithful to what was actually clicked, but a less reviewer-friendly timeline than structured mode.

---

## Mode 1, structured (`CLAUDE.md`) — iOS

[**Open the recording →**](https://app.revyl.ai/sessions/7e5fb852-ca67-4111-9aa8-d7e9fc5815ec)

```markdown
## 📱 Mobile PR Review

**Changes detected:** Removes the bug logic in `CartContext.tsx` that incorrectly swapped Orchid Mantis (product ID 3) for Gold Tortoise (product ID 4) when adding to cart.

**Test plan:**
1. Navigate to the Shop tab
2. Tap the Orchid Mantis product card
3. Verify product detail shows Orchid Mantis at $62.00
4. Tap ADD TO CART
5. Open cart and verify the correct product was added

**Result:** ❌ **Bug reproduced** — cart contains Gold Tortoise at $18.00 instead of Orchid Mantis at $62.00

> **Note:** This iOS simulator build predates the fix in this PR, so the bug is expected. The validation confirms the cart swap behavior that this PR removes.

**Session recording:** [View full recording and step timeline](https://app.revyl.ai/sessions/7e5fb852-ca67-4111-9aa8-d7e9fc5815ec)

_Tested on Revyl cloud device · iOS · build `398473a5`_
```

**Run stats:** 20 tool calls / 7.2 min / $0.28 / Sonnet 4.5  
**Revyl device commands:** `instruction` ×6, `validation` ×4, `start` ×3, `stop` ×2, `screenshot` ×1

This run survived a mid-session expiry (caught it, called `revyl device stop`, restarted via `revyl device start`, continued the test) without any human intervention. That's the durability you want from a PR review bot in production.

---

## Mode 1, reactive (`CLAUDE-reactive.md`) — iOS

[**Open the recording →**](https://app.revyl.ai/sessions/8ad07bc2-0aba-4026-a14d-15b80e355977)

```markdown
## 📱 Mobile PR Review (reactive)

**Changes detected:** The PR removes the product swap bug in `CartContext.tsx` where adding Orchid Mantis (product ID 3) to cart was incorrectly swapping it for Gold Tortoise (product ID 4).

**Hypothesis:** When adding Orchid Mantis ($62.00) to cart, the cart should contain Orchid Mantis at $62.00, not Gold Tortoise at $18.00.

**What I observed:** I navigated to the Orchid Mantis product detail page, confirmed it showed the correct product name and $62.00 price, then tapped ADD TO CART. A cart modal appeared at the bottom of the screen showing Gold Tortoise at $18.00 instead of the expected Orchid Mantis at $62.00. This successfully reproduces the swap bug that this PR aims to fix.

**Result:** ❌ Bug reproduced (as expected for pre-fix build)

**Expected:** Cart contains Orchid Mantis at $62.00  
**Observed:** Cart contains Gold Tortoise at $18.00

**Session recording:** [View full recording](https://app.revyl.ai/sessions/8ad07bc2-0aba-4026-a14d-15b80e355977)

_Tested on Revyl `ios` cloud device · build `398473a5` · reactive mode_
```

**Run stats:** 32 tool calls / 4.6 min / $0.49 / Sonnet 4.5  
**Revyl device commands:** `screenshot` ×8, `tap` ×7, `swipe` ×1, `start`/`stop` ×2/×1, **`instruction`/`validation` ×0**

Same reactive flow as Android, plus a `swipe` for scrolling to the Orchid Mantis card. Pure vision loop on iOS with no fallbacks.

---

## Cost / time summary

| | Structured | Reactive |
|---|---|---|
| **Android** | $0.68 / 4.6 min / 24 calls | $0.45 / 4.3 min / 30 calls |
| **iOS** | $0.28 / 7.2 min / 20 calls | $0.49 / 4.6 min / 32 calls |
| **Default workflow on PR open** | — | **$0.94 / ~5 min wall clock** for both platforms in parallel |

For ~$1 and 5 minutes of wall clock per PR, you get two real cloud-device runs (one Android, one iOS) producing two recordings reviewers can open in one click.
