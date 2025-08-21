---
sort: 3
---

# 課題の解答

以下は、課題の解答例です。
課題に合格されなかった方は、こちらのコードを参考にしてください。

**classes/order.php**

```php
<?php
// スーパークラスであるDbDataを利用するため
require_once __DIR__ . '/dbdata.php';

class Order extends DbData
{
    // カート内の全ての商品を注文内容として登録する
    public function addOrder(array $cartItems): void
    {
        // 注文テーブルに登録
        $sql = "INSERT INTO orders(orderdate) VALUES( ? )";
        $result = $this->exec($sql,  [date("Y-m-d H:i:s")]);
        // 注文番号を取得する
        $sql = "SELECT  last_insert_id( ) FROM orders";
        $stmt = $this->query($sql,  []);
        $result = $stmt->fetch();
        $orderId = $result[0];
        // 注文明細テーブルに登録する
        foreach ($cartItems  as  $item) {
            $sql = "INSERT INTO orderdetails VALUES( ?, ?, ? )";
            $result = $this->exec($sql,  [$orderId,  $item['ident'],  $item['quantity']]);
        }
    }
}
```

**order/order_now.php**

```php
<?php
// Cartオブジェクトを生成する
require_once __DIR__ . '/../classes/cart.php';
$cart = new Cart();
// カート内の全ての商品を取り出す
$cartItems = $cart->getItems();
// Orderオブジェクトを生成する
require_once __DIR__ . '/../classes/order.php';
$order = new Order();
// カート内の全ての商品を注文内容として登録する
// 注文テーブルordersと注文詳細テーブルorderdetailsに注文内容を登録する
$order->addOrder($cartItems);
?>
<!DOCTYPE html>
<html lang="ja">

<head>
    <meta charset="UTF-8">
    <title>ショッピングサイト</title>
    <link rel="stylesheet" href="../css/minishop.css">
</head>

<body>
    <h3>注文</h3>
    <p>お買い上げありがとうございました。<br>
        またのご利用をお待ちしております。</p>
    <table>
        <tr>
            <th>&nbsp;</th>
            <th>商品名</th>
            <th>メーカー・著者<br>アーティスト</th>
            <th>価格</th>
            <th>注文数</th>
            <th>金額</th>
        </tr>
        <?php
        $total = 0;
        foreach ($cartItems as $item) {
            $total += $item['price'] * $item['quantity'];
        ?>
            <tr>
                <td class="td_mini_img"><img class="mini_img" src="../images/<?= $item['image'] ?>"></td>
                <td class="td_item_name"><?= $item['name'] ?></td>
                <td class="td_item_maker"><?= $item['maker'] ?></td>
                <td class="td_right">&yen;<?= number_format($item['price']) ?></td>
                <td class="td_right"><?= $item['quantity'] ?></td>
                <td class="td_right">&yen;<?= number_format($item['price'] * $item['quantity']) ?></td>
            </tr>
        <?php
        }
        ?>
        <tr>
            <th colspan="5">合計金額</th>
            <td class="td_right">&yen;<?= number_format($total) ?></td>
        </tr>
    </table>
    <br>
    <a href="../index.php">ジャンル選択に戻る</a>
    <?php
    // カート内の全ての商品を削除する
    $cart->clearCart();
    ?>
</body>

</html>
```

**classes/cart.php** (clearCartメソッドの追加)

```php
<?php

// 

// カート内のすべての商品を削除する
  public function clearCart(): void
  {
    $sql = "DELETE FROM cart";
    $result = $this->exec($sql, []);
  }
```