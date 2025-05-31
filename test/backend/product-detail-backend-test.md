# product-detail-backend-test

## 1. Controller Test
| ID | Purpose | Input | Expect |
|----|---------|-------|--------|
| **CT‑01** | Basic rendering (login state irrelevant) | HTTP **GET** `/product/f9c9cfb2-…5274` | HTTP **200 OK**<br>View = `product-detail`<br>`model.dto` が存在 |
| **CT‑02** | Add to cart — cart absent | POST /api/cartParams: productId=<UUID>, qty=2Session: (none) | 200 OKsession.CART は Cart インスタンスproductService.addToCart() が 1 回呼ばれる |
| **CT‑03** | Add to cart — cart present | POST /api/cartSessionAttr: CART=<Cart>Params: productId=<UUID>, qty=2 | 200 OKsession.CART は渡した Cart と same instanceproductService.addToCart() が 1 回呼ばれる |

### qty パラメータ
| ID | qty | Expect |
|----|-----|--------|
| **Q‑01** | 0  | 400 |
| **Q‑02** | 1  | 200 |
| **Q‑03** | 10 | 200 |
| **Q‑04** | 20 | 200 |
| **Q‑05** | 21 | 400 |

---

## 2. Service Test 

| ID | Purpose | Input | Expect |
|----|---------|-------|--------|
| **ST‑01** | Guest flow | `productId = "f9c9cfb2-…5274"`<br>`loginUid = null` | DTO 値：<br>`productName = "Item18"`<br>`productDescription = "Item18の商品説明です。"`<br>`price = 3200`<br>`ratingAvg = 0.0`<br>`reviewCount = 0`<br>`isFav = false`<br>`outOfStock = false` |
| **ST‑02** | Logged‑in flow | `productId = "f9c9cfb2-…5274"`<br>`loginUid = "550e8400-e29b-41d4-a716-446655420000"` | DTO 値：<br>`productName = "Item18"`<br>`productDescription = "Item18の商品説明です。"`<br>`price = 3200`<br>`ratingAvg = 4.3 or 4.7` ※レビュー内容に合わせて可変<br>`reviewCount = 3`<br>`isFav = true`<br>`outOfStock = false` |
| **ST‑03** | 新規 | Cart 空 → add("P01", 3) 現行価格 1000 円 | productId="P01", qty=3, priceExTax=1000 |
| **ST‑04** | 価格上書 | Cart: P01 qty=5 priceEx=1000 → add("P01", 4) 現行価格 1200 円| productId="P01", qty=9, priceExTax=1200 |
| **ST‑05** | 上限丸め | Cart: P01 qty=15 priceEx=1200 → add("P01", 10) (価格 1200)| productId="P01", qty=20, priceExTax=1200 |
| **ST‑06** | 商品なし | 存在しないproductId　| ResponseStatusException |

---

## 3. Mapper Test

| ID | Purpose | Input | Expect |
|----|---------|-------|--------|
| **MD‑01** | Logged‑in user (ratings **4,4,5**) | `productId = "f9c9cfb2-…5274"`<br>`loginUid = random UUID`<br>`favorite` 行あり<br>`review` 3 件 (4,4,5) | `productName = "Item18"`<br>`productDescription = "Item18の商品説明です。"`<br>`price = 3200`<br>`ratingAvg = 4.3`<br>`reviewCount = 3`<br>`isFav = true`<br>`outOfStock = false` |
| **MD‑02** | Logged‑in user (ratings **4,5,5**) | 同上／`review` 3 件 (4,5,5) | `productName = "Item18"`<br>`productDescription = "Item18の商品説明です。"`<br>`price = 3200`<br>`ratingAvg = 4.7`<br>`reviewCount = 3`<br>`isFav = true`<br>`outOfStock = false` |
| **MD‑03** | Guest access (no reviews) | `productId = "f9c9cfb2-…5274"`<br>`loginUid = null`<br>`favorite` 行なし<br>`review` 0 件 | `productName = "Item18"`<br>`productDescription = "Item18の商品説明です。"`<br>`price = 3200`<br>`ratingAvg = 0.0`<br>`reviewCount = 0`<br>`isFav = false`<br>`outOfStock = false` |
| **MD‑04** | 旧価格取得 | id = `1e7b4cd6...` | priceExTax = 750 |

