# cart-backend-test

## 1. Controller Test
| ID | Purpose | Input | Expect |
|----|---------|-------|--------|
| **CT‑01** | カートが空の場合に `/cart` を表示できるか | GET `/cart` (セッションに `CART` 無し **または** 中身が空の `Cart`) | HTTP 200 / View 名 = `cart` / `Model` に `dto` 属性が存在 |
| **CT‑02** | カートに商品が存在する場合に `/cart` を表示できるか | GET `/cart` + `MockHttpSession` に `CART` = `Cart` { Pxx × qty… } を設定 | HTTP 200 / View 名 = `cart` / `Model` に `dto` 属性が存在 |
| **CT‑03** | **削除 API 正常系** — セッションあり & 削除対象 ID が存在 | `MockHttpSession` に `CART={ P02×1, P03×1 }` | HTTP **200** /  `data.items` に **P02 が含まれない** |
| **CT‑04** | **削除 API タイムアウト** — セッション無し | `session` なし | HTTP **200** / `data` が空 |

---

## 2. Service Test 

| ID | Purpose | Input | Expect |
|----|---------|-------|--------|
| **ST‑01** | カート空 | Cart: items = {} | CartView.lines = []　totalQty = 0　totalPayment = 0 |
| **ST‑02** | 	4 行混在表示① 不変  ② 価格改定  ③ 販売終了  ④ 販売再開 | **P01** qty=2 lastInc=1100 (販売中・価格据え置き)<br>**P02** qty=1 lastInc=1000 (DB 価格 1200 → 値上げ)<br>**P03** qty=4 lastInc=500  (DB onSale=false → 販売終了)<br>**P04** qty=3 lastInc=800  (Cart onSale=false, DB onSale=true → 販売再開)  | **lines.size = 4**<br><br>• **P01**: `onSale=true`, `priceChanged=false`, `subtotal=2,200`<br>• **P02**: `onSale=true`, `priceChanged=true`,  `subtotal=1,200`<br>• **P03**: `onSale=false`, `subtotal=0`<br>• **P04**: `onSale=true`, `priceChanged=false`, `subtotal=2,400`<br><br>`totalQty = 6` (販売中のみ ※2+1+3)<br>`totalPayment = 5,800`  |
| **ST‑03** | 削除(IDが存在しない) |Cart　● P01 qty=1,lastInc=100 | CartView.lines = []　totalQty = 0　totalPayment = 0 |
| **ST‑04** | 削除(ID存在) | 前提 Cart　● P01 qty=2,lastInc=1100　● P02 qty=1, lastInc=1000 (削除対象)　● P03 qty=4, lastInc=500　操作 deleteItem(cart,"P02") |アイテム数、お支払金額、各カートアイテムの内容が不変 |


## 4. 保留・後回しメモ 

| 項目 | 意味・背景 | 
|------|-----------|
| **バリデーションチェック** | 削除時のproductId | 

