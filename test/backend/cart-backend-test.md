# cart-backend-test

## 1. Controller Test
| ID | Purpose | Input | Expect |
|----|---------|-------|--------|
| **CT‑01** | カートが空の場合に `/cart` を表示できるか | GET `/cart` (セッションに `CART` 無し **または** 中身が空の `Cart`) | HTTP 200 / View 名 = `cart` / `Model` に `dto` 属性が存在 |
| **CT‑02** | カートに商品が存在する場合に `/cart` を表示できるか | GET `/cart` + `MockHttpSession` に `CART` = `Cart` { Pxx × qty… } を設定 | HTTP 200 / View 名 = `cart` / `Model` に `dto` 属性が存在 |

---

## 2. Service Test 

| ID | Purpose | Input | Expect |
|----|---------|-------|--------|
| **ST‑01** | カート空 | Cart: items = {} | CartView.lines = []　totalQty = 0　totalPayment = 0 |
| **ST‑02** | カート3行混在表示 | Cart 内:P01 qty=2 lastInc=1100 (販売中・価格据え置き)　P02 qty=1 lastInc=1000 (DB 価格 1,200 → priceChanged = true)　P03 qty=4 lastInc=500 (DB onSale=false → onSale = false) |lines.size() == 3　lines[0].onSale == true, priceChanged == false, subtotal = 2200　lines[1].onSale == true, priceChanged == true, subtotal = 1200　lines[2].onSale == false, subtotal = 0　totalQty = 2 + 1 = 3 (販売中のみ) |


