# カート画面 ― 設計

## 1. 画面仕様

| 要素 | 内容 |
|------|-------------|
| **URL** | `/cart` |
| **アイテム総数** | 画面ヘッダに **「アイテム数 ○点」**  |
| **サムネイル** | 画像 1 枚（販売終了品は 50 % 透過） |
| **商品情報** | 商品名 / **税込単価** // 数量 / **税込小計**<br>旧税込単価は `priceChanged` が `true` の行のみ表示し、<br>赤文字＋取り消し線（例 `（旧 ¥350）`）。<br>販売終了品は行グレーアウト＋`販売終了`バッジ、小計は **¥0** |
| **数量選択** | 1–20 ＋ ±ボタン（販売終了品は `disabled`） |
| **削除** | 「削除」リンク |
| **お支払金額** | **商品計（税込）**＝`CartDto.totalPrice` |
| **空カート** | 行リストが空なら「カートに商品がありません」画面＋“買い物を続ける”ボタン |

> *税込単価・旧税込単価・税込小計・総額は **DTO 内で計算済み**。テンプレート側は値を並べるだけで OK。*

## 2. バリデーションチェック

## 3. 処理フロー
### 3-1. 初期表示
1. **リクエスト受付** — `GET /cart`
2. **セッションからカート取得** — 空なら `CartDto.empty()` を返して終了。
3. **商品マスタ一括取得**  
   ```sql
   SELECT
       p.product_id,
       p.product_name,
       p.price,
       p.sale_status
   FROM product p
   WHERE p.product_id IN (:ids);
   ```

<a id="cartitemdto"></a>
4. **CartItemDto 生成・旧価格比較**  
   * 税込単価 = `TaxUtil.inc(price)`。  
   * `prevPrice`（前回表示価格）をセッションから取得。  
   * `priceChanged = ( prevPrice != priceIncTax)`  
   * 行 DTO に `prevPrice` と `priceChanged` をセット。  
   * 行小計 = `priceIncTax × qty`（販売終了 or priceChanged 行でも 0 円計算ルールは同じ）。  
   * 販売中の商品のみ、`prevPrice = priceIncTax` をセッションへ保存。  

5. **CartViewDto 生成（コンストラクタ一括計算）**  
   合計点数・総額を 1 回のストリームで算出しフィールド固定。 

### 3-2. 削除
1. **リクエスト受付** — `POST /api/cart/delete/{productId}`
2. **セッションからカート取得** — 取得できない場合は メッセージ「カートの有効期限が切れました。」 を返す     
3. **カートから削除**  - `cart.getItems().remove(productId);  ` 
4. **CartViewDto 生成**  - [CartItemDto 生成・旧価格比較](#cartitemdto) を参照
5. **レスポンス返却** — 200と メッセージ「削除しました」 を返す
 
### 3-3. 数量変更


## 4. 保留・後回しメモ 

| 項目 | 意味・背景 | 
|------|-----------|
| **商品マスタ一括取得クエリ** | SELECT(*)でどれだけ負荷あるか | 
| **Cart implements Serializable** | implements Serializable付与しないで何が困るか | 
| **スレッドセーフ** | 複数タブで操作したときの整合性 |

