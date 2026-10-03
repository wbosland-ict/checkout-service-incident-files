# PR #4821 — "Refactor cart items retrieval to lazy-loaded ORM associations"

**Merged:** 2026-07-07 09:41 UTC — **Released as:** checkout-service v2.14.0
**Reviewers:** 1 approval (J. Alvarez). CI checks: unit tests ✅, integration
tests ✅, lint ✅. No performance/load test stage in pipeline.

**PR description (excerpt):**
> Removes ~80 lines of manual mapping code. The ORM now loads cart items and
> their products through the entity associations, which is cleaner and
> easier to maintain. Needed as groundwork for the "saved carts" feature
> (v2.15.0).

## Diff excerpt

```diff
--- a/src/main/java/com/shopfast/checkout/cart/CartRepository.java
+++ b/src/main/java/com/shopfast/checkout/cart/CartRepository.java
@@ -18,22 +18,9 @@ public interface CartRepository extends JpaRepository<Cart, Long> {

-    @Query("""
-        SELECT c FROM Cart c
-        JOIN FETCH c.items i
-        JOIN FETCH i.product p
-        WHERE c.id = :cartId
-        """)
-    Optional<Cart> findByIdWithItemsAndProducts(@Param("cartId") Long cartId);
+    Optional<Cart> findById(Long cartId);
```

```diff
--- a/src/main/java/com/shopfast/checkout/cart/CartItem.java
+++ b/src/main/java/com/shopfast/checkout/cart/CartItem.java
@@ -31,7 +31,7 @@ public class CartItem {

-    @ManyToOne(fetch = FetchType.EAGER)
+    @ManyToOne(fetch = FetchType.LAZY)
     @JoinColumn(name = "product_id")
     private Product product;
```

```diff
--- a/src/main/java/com/shopfast/checkout/CheckoutService.java
+++ b/src/main/java/com/shopfast/checkout/CheckoutService.java
@@ -52,14 +52,12 @@ public class CheckoutService {

+    // Needed so lazy associations can be loaded (open_in_view is disabled)
+    @Transactional
     public CheckoutResult start(Long cartId) {
-        List<LineItem> lines = cartReader.readLineItems(cartId); // @Transactional(readOnly), connection released on return
+        Cart cart = cartRepository.findById(cartId).orElseThrow(CartNotFoundException::new);
+        List<LineItem> lines = cart.getItems().stream()
+            .map(i -> new LineItem(i.getProduct().getSku(), i.getProduct().getPrice(), i.getQuantity()))
+            .toList();
         Reservation reservation = inventoryClient.reserve(lines);
         PaymentResult payment = paymentClient.charge(reservation, lines);
         return orderWriter.create(cartId, reservation, payment);
     }
```

```diff
--- a/src/main/java/com/shopfast/checkout/cart/CartReader.java
+++ /dev/null
-@Component
-class CartReader {
-    @Transactional(readOnly = true)
-    List<LineItem> readLineItems(Long cartId) {
-        Cart cart = cartRepository.findByIdWithItemsAndProducts(cartId)
-            .orElseThrow(CartNotFoundException::new);
-        return cartMapper.toLineItems(cart);   // manual mapping
-    }
-}
```

## Test data note (from the integration test suite)

Integration tests use fixture carts with **1–2 items**. Production carts at
peak average **4.6 items** (p95: 11 items).
