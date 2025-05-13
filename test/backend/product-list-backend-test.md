# product-list-backend-test

## 1. Controller Test

### page パラメータ
| ID | page | Expect |
|----|------|---------------|
| P-01 | null | 1 |
| P-02 | 1 | 1 |
| P-03 | 2 | 1 |
| P-04 | 0 | 1 |
| P-05 | abc | 1 |

### sort パラメータ
| ID | sort | Expect |
|----|------|--------|
| S-01 | null | NEW |
| S-02 | NEW  | NEW |
| S-03 | HIGH | HIGH |
| S-04 | LOW  | LOW |
| S-05 | foo  | NEW |

### q (keywords) パラメータ
| ID | q 値 | Expect |
|----|------|--------|
| Q-01 | null | empty |
| Q-02 | 空白で区切られた2語 | 2 words |
---

## 2. Service Test 

| ID | Test Case | Input (page/sort/keywords) | Assertions |
|----|-----------|---------------------------|------------|
| S-01 | ProductListDto fields mapping | **1 / NEW / keywordあり | - `products` サイズ<br>- `currentPage` 値<br>- `pageNumbers` 一致<br>- **`products[*].productId`**<br>- **`products[*].productName`**<br>- **`products[*].price`**<br>- **`products[*].outOfStock`**  |

---

## 3. Mapper Test

| ID | Purpose | Input | Expect |
|----|---------|-------------|--------|
| M-01 | Paging (page=1) | page=1 | OFFSET 0 |
| M-02 | Paging (page=2) | page=2 | OFFSET pageSize |
| M-03 | sort HIGH | sort=HIGH | price DESC |
| M-04 | sort LOW | sort=LOW | price ASC |
| M-05 | sort NEW | sort=NEW | created_at DESC |
| M-06 | SecondarySort productId | sort=NEW | product_id DESC |
| M-06 | keywords none | empty | no WHERE keyword |
| M-07 | keywords present | keywords=[kw1, kw2] | いずれかのキーワードに一致 |
| M-08 | stock flag (in‑stock) | - | outOfStock=false |
| M-09 | stock flag (out‑of‑stock) | - | outOfStock=true |
