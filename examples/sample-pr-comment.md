## 📱 Mobile PR Review

> PR: **fix: Orchid Mantis adds wrong product to cart**

**Changes detected:** Removed the cart substitution bug in `CartContext.tsx` — the `addToCart` function previously swapped Orchid Mantis (id:3) for Gold Tortoise (id:4) before adding to cart. The fix drops the `product.id === 3` conditional and uses the original product directly.

```diff
  const addToCart = useCallback((product: Product) => {
-   // BUG: Adding Orchid Mantis (id:3) silently adds Gold Tortoise (id:4) instead
-   const actualProduct = product.id === 3
-     ? allProducts.find(p => p.id === 4)!
-     : product;
-
    setItems(prev => {
-     const existing = prev.find(item => item.id === actualProduct.id);
+     const existing = prev.find(item => item.id === product.id);
```

**Test plan:**
1. Open the Shop tab and confirm the product grid renders
2. Scroll to Orchid Mantis and tap the card
3. Tap ADD TO CART on the product detail page
4. Open the cart and validate Orchid Mantis is present at $62.00

**Result:** ❌ Bug reproduced on the pre-fix build — the cart contained **Gold Tortoise at $18.00** after adding Orchid Mantis. Step 4 (`validation: The cart contains Orchid Mantis at $62.00`) failed.

**Session recording:** [View full recording and step timeline](https://app.revyl.ai/sessions/sess_01hfxyzabcdef)

_Tested on Revyl cloud device · build `bv_7a3b2c1d` · re-run with `/review`_
