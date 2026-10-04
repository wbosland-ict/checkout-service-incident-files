# PR #4821 — "Refactor cart items retrieval to lazy-loaded EF Core navigation properties"

**Merged:** 2026-07-07 09:41 UTC — **Released as:** checkout-service v2.14.0
**Reviewers:** 1 approval (J. Alvarez). CI checks: unit tests ✅, integration
tests ✅, lint ✅. No performance/load test stage in pipeline.

**PR description (excerpt):**
> Removes ~80 lines of manual mapping code. EF Core now loads cart items and
> their products through the navigation properties, which is cleaner and
> easier to maintain. Needed as groundwork for the "saved carts" feature
> (v2.15.0).

## Diff excerpt

```diff
--- a/src/ShopFast.Checkout/Cart/CartRepository.cs
+++ b/src/ShopFast.Checkout/Cart/CartRepository.cs
@@ -15,22 +15,9 @@ public class CartRepository : ICartRepository
 {
     private readonly CheckoutDbContext _dbContext;

-    public async Task<Cart?> GetByIdWithItemsAndProductsAsync(long cartId)
-    {
-        return await _dbContext.Carts
-            .Include(c => c.Items)
-                .ThenInclude(i => i.Product)
-            .FirstOrDefaultAsync(c => c.Id == cartId);
-    }
+    public async Task<Cart?> GetByIdAsync(long cartId)
+    {
+        return await _dbContext.Carts.FindAsync(cartId);
+    }
 }
```

```diff
--- a/src/ShopFast.Checkout/Cart/CartItem.cs
+++ b/src/ShopFast.Checkout/Cart/CartItem.cs
@@ -28,7 +28,8 @@ public class CartItem
     public int Quantity { get; set; }

     public long ProductId { get; set; }
-    public Product Product { get; set; }
+    // Lazy-loading proxy: fetched on first access instead of being
+    // eager-loaded by the repository query (requires UseLazyLoadingProxies()).
+    public virtual Product Product { get; set; }
 }
```

```diff
--- a/src/ShopFast.Checkout/CheckoutService.cs
+++ b/src/ShopFast.Checkout/CheckoutService.cs
@@ -49,14 +49,15 @@ public class CheckoutService
 {

+    // Needed so lazy-loaded navigation properties can still be read;
+    // keeps the DbContext (and its DB connection) open for the whole method.
     public async Task<CheckoutResult> StartAsync(long cartId)
     {
-        var lines = await _cartReader.ReadLineItemsAsync(cartId); // short-lived scope, connection released on return
+        using var transaction = await _dbContext.Database.BeginTransactionAsync();
+        var cart = await _cartRepository.GetByIdAsync(cartId)
+            ?? throw new CartNotFoundException();
+        var lines = cart.Items
+            .Select(i => new LineItem(i.Product.Sku, i.Product.Price, i.Quantity))
+            .ToList();
         var reservation = await _inventoryClient.ReserveAsync(lines);
         var payment = await _paymentClient.ChargeAsync(reservation, lines);
-        return await _orderWriter.CreateAsync(cartId, reservation, payment);
+        var result = await _orderWriter.CreateAsync(cartId, reservation, payment);
+        await transaction.CommitAsync();
+        return result;
     }
 }
```

```diff
--- a/src/ShopFast.Checkout/Cart/CartReader.cs
+++ /dev/null
-public class CartReader
-{
-    private readonly ICartRepository _cartRepository;
-
-    public async Task<List<LineItem>> ReadLineItemsAsync(long cartId)
-    {
-        // Own short-lived DbContext scope; connection released as soon as
-        // the mapped line items are returned.
-        var cart = await _cartRepository.GetByIdWithItemsAndProductsAsync(cartId)
-            ?? throw new CartNotFoundException();
-        return CartMapper.ToLineItems(cart);   // manual mapping
-    }
-}
```

## Test data note (from the integration test suite)

Integration tests use fixture carts with **1–2 items**. Production carts at
peak average **4.6 items** (p95: 11 items).
