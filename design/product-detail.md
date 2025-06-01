# 商品詳細画面 ― 設計

## 1. 画面仕様

| 要素 | 内容 |
|------|------|
| URL | `/product/{productId}` |
| サムネイル | 画像1枚 |
| 在庫バッジ | `在庫切れ` 灰色 |
| 商品情報 | 商品名、価格、説明 |
| レビュー | 平均値、件数 |
| お気に入り | toggle |
| カートに入れる | 新着、価格が安い、価格が高い |
| 数量選択 | 1–20 + ±ボタン |
| 在庫切れ時の UI ルール | 在庫切れバッジ表示、 カートボタン & 数量入力を非活性|

## 2. バリデーションチェック

## 3. 処理フロー
### 3-1. 初期表示
1. **アクセス受付**  
   - 例: `/product/{productId}`  

2. **ログインユーザーID取得**  
   - Spring Security で Authentication#getPrincipal() から userId を取り出す。  
   - 未ログイン時は userId = null として後続へ渡す。

3. **SQL 構築**  

```xml
SELECT
    p.product_name,
    p.description,
    p.price             AS price,
    (p.stock_qty > 0)            AS in_stock,

    /* ── レビュー集計 ───────────────── */
    COALESCE(rs.avg_score, 0)    AS rating_avg,
    COALESCE(rs.review_cnt, 0)   AS review_cnt,

    /* ── お気に入りフラグ ────────────── */
    <choose>
      <when test="userId != null">
        CASE WHEN f.user_id IS NULL THEN 0 ELSE 1 END AS is_favorite
      </when>
      <otherwise>
        0 AS is_favorite   <!-- 未ログイン時は固定 false -->
      </otherwise>
    </choose>

FROM product p

<!-- ログインユーザが存在するときだけ favorite を結合 -->
<if test="userId != null">
  LEFT JOIN favorite f
         ON f.product_id = p.product_id
        AND f.user_id    = #{userId}
</if>

WHERE p.product_id = #{productId};
```

4. **DTO 返却**  
   - 商品情報、レビュー情報をセット。
   - 税込み価格を計算。 

### 3-2. お気に入り追加/解除
1. **リクエスト**  
   - 追加: `/product/{product_id}/favorite/add`
   - 削除: `/product/{product_id}/favorite/delete` 

2. **CSRF対策**  
CSRF トークンを付与

3. **未ログイン時の挙動**  
ログイン画面へリダイレクト

1. **DB処理**  
`userId`と`productId`をもとに、お気に入りテーブルを更新(登録or削除) 

1. **レスポンス**  
ステータスコードに応じて、メッセージ表示

### 3-3. 数量選択  
1. **バリデーションチェック**  
   1 ≤ qty ≤ 20か判定。20に達した場合はメッセージ

2. **活性状態の切り替え**  
qty <= 1 →マイナスボタン非活性  
qty >= 20 →プラスボタン非活性


### 3-4. カートに入れる
1. **AJAX通信**  
`POST /api/cart` に {productId, qty} を JSON 送信。CSRF トークンをヘッダで付与

2. **バリデーションチェック**  
@Min(1) @Max(20) で検証

3. **カート更新**  
   - DBからpriceを取得し、旧価格として保持
   - セッションから Cart を取得（無ければ生成）
   - merged = min(existing + qty, 20) を計算し cart.put(productId, merged, price);

4. **JSON レスポンス**  
ステータスコードに応じてメッセージ表示


## 4. 保留・後回しメモ 

| 項目 | 意味・背景 | 
|------|-----------|
| **星（★）表示** | 初期リリースでは数値のみ表示し、デザイン確定後に ★ アイコン＋0.1 刻み描画を追加 | 
| **クエリ JOIN 最適化 & 計測** | 暫定ですべての詳細クエリを JOIN で一発取得する設計 |
| **comment-varchar(500)** | 暫定で500文字以内 |
| **findProductDetail** | p.product_id１カラム増えるだけでどれだけ負荷増えるか |
| **旧価格取得** | 毎回旧価格保持して、どれだけ負荷かかるか　SELECT(*)でどれだけ負荷かかるか |
| **スレッドセーフ** | 複数タブで操作したときの整合性 |