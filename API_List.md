# お知らせ

## `add_notification_public($content, $sent_at, $sent_by, $icon)`

【管理画面】手動お知らせ作成（送信）

### Query

```graphql
mutation add_notification_public(
  $sent_by: Int! # 管理者ID (users table pk)
  $sent_at: date! # 送信日時
  $content: String! # 送信内容
  $icon: String
) {
  insert_notifications_one(
    object: {
      sent_by: $sent_by # 管理者ID (users table pk)
      sent_at: $sent_at # 送信日時
      content: $content # 送信内容
      icon: $icon
      type: 0 # 0: public（手動通知）, 1: private（自動通知）
    }
  ) {
    id # notifications table pk
    content # 通知内容
    icon
    sent_at # 送信日時
    sender {
      id # 管理者ID (users table pk) (admin)
      name # 送信者名
    }
    type # 通知タイプ（手動：０，自動：１）
  }
}
```

### Response

```graphql
{
  data: {
    insert_nofitications_one: {
      id: Int # notifications table pk
      content: String # 通知内容
      icon: String
      sent_at: date # 送信日時
      sender: {
        id: Int # 管理者ID (users table pk)
        name: String # 送信者名
      }
      type: Int # 通知タイプ（手動：０，自動：１）
    }
  }
}
```

## `update_notification_public($notification_id, $read_by)`

【アプリ用】手動お知らせ更新（送信）

### Query

```graphql
mutation update_notification_public (
  $notification_id: Int! # notifications table pk
  $read_by: jsonb! # reader's ID
) {
  update_notifications_by_pk (
    pk_columns: { id: $notification_id } # notifications table pk
    _append: {
      readers: $read_by
    }
  ) {
    id # notifications table pk
    content # 通知内容 updated
    sent_at # 送信日時
    sender {
      id # 管理者ID (users table pk)
      name # 送信者名
    }
    type # 通知タイプ（手動：０，自動：１） 0: public, 1: private
    readers # Array of readers
  }
}
```

### Response

```graphql
{
  data: {
    update_notifications_by_pk: {
      id: Int # notifications table pk
      content: String # 通知内容
      sent_at: date # 送信日時
      sender: {
        id: Int # 管理者ID (users table pk)
        name: String # 送信者名
      }
      type: Int # 通知タイプ（手動：０，自動：１）
      readers: jsonb # array of readers
    }
  }
}
```

## `get_notification_public($user_id)`

【管理画面】, 【アプリ用】手動お知らせ履歴取得

### Query

```graphql
get_notification_public (
  $user_id: jsonb # reader's ID
) {
  read: notifications (
    where: {
      type: { _eq: 0 },
      readers: { _contains: $user_id}
    }
  ) {
    id # notifications table pk
    content # 通知内容
    icon
    sent_at # 送信日時
    sender {
      id # 管理者ID (users table pk)
      name # 送信者名
    }
    type # 通知タイプ（手動：０，自動：１）
  }
  unread: notifications (
    where: {
      type: { _eq: 0 },
      _not: { readers: { _contains: $user_id}}
    }
  ) {
    id # notifications table pk
    content # 通知内容
    icon
    sent_at # 送信日時
    sender {
      id # 管理者ID (users table pk)
      name # 送信者名
    }
    type # 通知タイプ（手動：０，自動：１）
  }
}
```

### Response

```graphql
{
  data: {
    read: [
      {
        id: Int # notifications table pk
        content: String # 通知内容
        icon: String
        sent_at: date # 送信日時
        sender: {
          id: Int # 管理者ID (users table pk)
          name: String # 送信者名
        }
        type: Int # 通知タイプ（手動：０，自動：１） 0: public, 1: private
      }
    ]
    unread: [
      {
        id: Int # notifications table pk
        content: String # 通知内容
        sent_at: date # 送信日時
        sender: {
          id: Int # 管理者ID (users table pk)
          name: String # 送信者名
        }
        type: Int # 通知タイプ（手動：０，自動：１） 0: public, 1: private
      }
    ]
  }
}
```

## `add_notification_private($recipient_id, $sent_by, $sent_at, $content)`

【管理画面】【アプリ用】自動お知らせ作成（送信)

### Query

```graphql
mutation add_notification_private (
  $recipient_id: Int! # ユーザーID
  $sent_by: Int # 管理者ID (users table pk)
  $sent_at: date! # 送信日時
  $content: String! # 送信内容
) {
  insert_notifications_one (
    object: {
      sent_by: $sent_by # 管理者ID (users table pk)
      sent_at: $sent_at # 送信日時
      content: $content # 送信内容
      recipient_id: $recipient_id # ユーザーID
      type: 1 # 0: public（手動通知）, 1: private（自動通知）
    }
  ) {
    id # notifications table pk
    recipient {
      id # 受信者ID
      name # 受信者名
    }
    sender {
      id # 管理者ID (users table pk)
      name # 送信者名
    }
    content # 通知内容
    sent_at # 送信日時
    type # 通知タイプ（手動：０，自動：１）
  }
}
```

### Response

```graphql
{
  data: {
    insert_notifications_one: {
      id: Int # notifications table pk
      recipient: {
        id: Int # 受信者ID
        name: String # 受信者名
      }
      sender: {
        id: Int # 管理者ID (users table pk)
        name: String # 送信者名
      }
      content: String # 通知内容
      sent_at: date # 送信日時
      type: Int # 通知タイプ（手動：０，自動：１）
    }
  }
}
```

## `get_notification_private($recipient_id, $date)`

【アプリ用】自動お知らせ履歴取得

### Query

```graphql
get_notification_private (
  $recipient_id: Int # ユーザーID
  $date: date # limit date
) {
  notifications (
    where: {
      recipient_id: { _eq: $recipient_id },
      sent_at: { _gt: $date }
    }
  ) {
    id # notifications table pk
    recipient {
      id # 受信者ID
      name # 受信者名
    }
    sender {
      id # 管理者ID (users table pk)
      name # 送信者名
    }
    content # 通知内容
    sent_at # 送信日時
    type # 通知タイプ（手動：０，自動：１）
  }
}
```

### Response

```graphql
{
  data: {
    notifications: [
      {
        id: Int # notifications table pk
        recipient: {
          id: Int # 受信者ID
          name: String # 受信者名
        }
        sender: {
          id: Int # 管理者ID (users table pk)
          name: String # 送信者名
        }
        content: String # 通知内容
        sent_at: date # 送信日時
        type: Int # 通知タイプ（手動：０，自動：１）
      }
    ]
  }
}
```

## `delete_notification($notification_id)`

【管理画面】ID によるお知らせせ削除

### Query

```graphql
mutation delete_notification (
  $notification_id: Int!
) {
  delete_notifications_by_pk (
    id: $notification_id
  ) {
    id # notifications table pk
    content # 通知内容
    sent_at # 送信日時
    sender {
      id # 管理者ID (users table pk)
      name # 送信者名
    }
    type # 通知タイプ（手動：０，自動：１）
  }
}
```

### Response

```graphql
{
  data: {
    delete_notifications_by_pk: {
      id: Int # notifications table pk
      content: String # 通知内容
      sent_at: date # 送信日時
      sender: {
        id: Int # 管理者ID (users table pk)
        name: String # 送信者名
      }
      type: Int # 通知タイプ（手動：０，自動：１）
    }
  }
}
```

<!-- # キーワード検索

## `search_items_by_keyword($keyword, $order_by)`

【アプリ用】 キーワードによる商品検索

### Data type

list_order_by

```graphql
{
  data_field: order
}
# example
# { id: asc, name: desc }
```

### Query

```graphql
searh_items_by_keyword (
  $keyword: String! # キーワード 例）'%keyword%'
  $order_by: list_order_by # 並べ替え（指定ないとデフォルトは id desc）
) {
  items (
    where: {
      name: { _like: $keyword } # キーワード
    }
    order_by: $order_by # 並べ替え
  ) {
      id # 商品ID
      name # 商品名
      images # 画像URL
      inventories {
        unit_price # 単価
      }
      producer {
        name # 生産者名
      }
      item_favorites {
        user_id # 当商品をお気に入りしているユーザーID
      }
  }
}
```

### Response

```graphql
{
  data: {
    items: [
      {
        id: Int # 商品ID
        name: String # 商品名
        images: [String] # 画像URL
        inventories: {
          unit_price: Int # 単価
        }
        producer: {
          name: String # 生産者名
        }
        item_favorites: [
          {
            user_id: Int # 当商品をお気に入りしているユーザーID
          }
        ]
      }
    ]
  }
} 
```-->

# 条件で絞り込み検索

## `list_items_filter_by_condition($keyword, $category, $ship_date, $address, $price_lower, $price_upper, $characteristic_id, $order_by)`

【アプリ用】条件で絞り込み検索

### Data type

items_order_by

```graphql
{
  < field name >: < order >
} # field for sorting 
# example
# { "id": "asc" }
```

### Query

```graphql
list_items_filter_by_condition (
  $keyword: String # キーワード 例）'%keyword%'
  $category_id: Int # カテゴリーID
  $ship_date: date # 到着希望日
  $address: jsonb # 産地  
  $price_lower: Int # 価格帯（低）
  $price_upper: Int # 価格帯（高）
  $characteristic_id: Int
  $order_by: [inventories_order_by!] # 並べ替え
) {
  inventories (
    where: {
      ship_date: {_eq: $ship_date}
      unit_price: {
        _gt: $price_lower # 価格帯（低）
        _lt: $price_upper # 価格帯（高）
      }
      item: {
        name: {_like: $keyword}
        category_id: {_eq: $category_id}
        producer: {
          address: {_contains: $address}
        }
        characteristic_id: {_eq: $characteristic_id}
      }
    },
    order_by: $order_by # 並べ替え
  ) {
    id
    item {
      id # 商品ID
      name # 商品名
      images # 画像URL
      producer {
        name # 生産者名
      }    
      item_favorites {
        user_id # このアイテムが好きなuser_id
      }
      characteristic {
        id
        name
        type
      }
      tax_rate
    }
    unit_price # 単価
    ship_date
    available_box_count
  }
}
```

### Response

```graphql
{
  data: {
    inventories: [
      {
        id
        item: {
          id: Int # 商品ID
          name: String # 商品名
          images: [String] # 画像URL
          producer {
            name: String # 生産者名
          }    
          item_favorites: {
            user_id: Int # このアイテムが好きなuser_id
          }
          characteristic: {
            id: Int
            name: String
            type: String
          }
          tax_rate: Float
        }
        unit_price: Int # 単価
        ship_date: date
        available_box_count: Int
      }
    ]
  }
}
```

# お気に入り

## `list_favorite_items_by_user($user_id, $order_by)`

【アプリ・管理画面用】ユーザーのお気に入り一覧取得

### Data type

item_favorite_order_by

```graphql
{
  < field name >: < order >
} 
# field for sorting
# example
# { id: asc, name: desc }
```

### Query

```graphql
list_favorite_items_by_user (
  $user_id: Int! # ユーザーID  
  $order_by: [item_favorite_order_by!] # 並べ替え
) {
  item_favorite (
    where: {
      user_id: { _eq: $user_id }
    }
    order_by: $order_by # 並べ替え
  ) {
    item {
      id # 商品ID
      name # 商品名
      images # 画像URL
      inventories {
        id
        unit_price # 単価
      }
      producer {
        name # 生産者名
      }
      characteristic {
        id
        name
        type
      }
    }
  }
}
```

### Response

```graphql
{
  data: {
    items: [
      {
        id: Int # 商品ID
        name: String # 商品名
        images: jsonb # 画像URL
        inventories: {
          id
          unit_price: Int # 単価
        }
        producer: {
          name: String # 生産者名
        }
        characteristic: {
          id: Int
          name: String
          type: String
        }
      }
    ]
  }
}
```

## `list_users_by_favorite_item($item_id)`

【管理画面】該当商品をお気に入り登録しているユーザ一覧取得

### Query

```graphql
list_users_by_favorite_item (
  $item_id: Int! # 商品ID
) {
  item_favorite (
    where: {
      item_id: {_eq: $item_id} # 商品ID
    }
  ) {
    users {
      id # ユーザーID
      name # ユーザー名
      email # メールアドレス
    }
  }
}
```

### Response

```graphql
{
  data: {
    users: [
      {
        id: Int # ユーザーID
        name: String # ユーザー名
        email: String # メールアドレス
      }
    ]
  }
}
```

## `add_favorite_item($user_id, $item_id)`

【アプリ用】お気に入り登録

### Query

```graphql
mutation add_favorite_item(
  $user_id: Int! # ユーザーID
  $item_id: Int! # 商品ID
) {
  insert_item_favorite_one(
    object: {
      user_id: $user_id # ユーザーID
      item_id: $item_id # 商品ID
    }
  ) {
    item {
      id # 商品ID
      name # 商品名
    }
  }
}
```

### Response

```graphql
{
  data: {
    insert_item_favorite_one: {
      item: {
        id: String # 商品ID
        name: String # 商品名
      }
    }
  }
}
```

## `remove_favorite_item($user_id, $item_id)`

【アプリ・管理画面用】お気に入り解除

### Query

```graphql
mutation remove_favorite_item(
  $user_id: Int! # ユーザーID
  $item_id: Int! # 商品ID
) {
  delete_item_favorite(
    where: {
      user_id: { _eq: $user_id } # ユーザーID
      item_id: { _eq: $item_id } # 商品ID
    }
  ) {
    affected_rows
  }
}
```

### Response

```graphql
{
  data: {
    affected_rows: Int
  }
}
```

# カート管理

## `add_cart_inventory($user_id, $inventory_id, $amount, $delivery_date, $sold_by)`

【アプリ】カートに商品追加

### Query

```graphql
mutation add_cart_inventory(
  $user_id: Int! # ユーザーID
  $inventory_id: Int!
  $amount: Int! # 商品数
  $delivery_date: date! # 配達日
) {
  insert_inventory_cart_one(
    object: {
      user_id: $user_id # ユーザーID
      inventory_id: $inventory_id # 商品ID
      amount: $amount # 商品数
      delivery_date: $delivery_date # 追加日時
    }
  ) {
    inventory {
      id
      item {
        id # 商品ID
        name # 商品名
        images # 画像URL
        producer {
          name
          email
          tel
          address
        }
        item_favorites {
          user_id # このアイテムが好きなuser_id
        }
        characteristic {
          id
          name
          type
        }
        tax_rate
      }
      unit_price # 単価
      ship_date
      available_box_count
    }
    amount # 商品数
    delivery_date # 配達日
  }
}
```

### Response

```graphql
{
  data: {
    insert_inventory_cart_one: {
      id: Int # item-cart table pk
      user_id: Int
      inventory: {
        id: Int
        item: {
          id: Int # 商品ID
          name: String # 商品名
          images: [String] # 画像URL
          producer {
            name: String # 生産者名
          }    
          item_favorites: {
            user_id: Int # このアイテムが好きなuser_id
          }
          characteristic: {
            id: Int
            name: String
            type: String
          }
          producer: {
            name: String
            email: String
            tel: String
            address: jsonb
          }
          tax_rate: Float
        }
        unit_price: Int # 単価
        ship_date: date
        available_box_count: Int
      }
      amount: Int # 商品数
      delivery_date: date # 配達日
    }
  }
}
```

## `remove_cart_inventory($cart_id)`

【アプリ用】カート内の商品削除

### Query

```graphql
mutation remove_cart_item(
  $cart_id: Int! # PK of `item_cart` table
) {
  delete_item_cart(
    where: {
      id: { _eq: $cart_id } # PK of `item_cart` table
    }
  ) {
    affected_rows
  }
}
```

### Response

```graphql
{
  data: {
    affected_rows: Int
  }
}
```

## `list_cart_inventories_by_user($user_id)`

【アプリ用】カート内の商品一覧取得

### Query

```graphql
list_cart_inventories_by_user (
  $user_id: Int! # ユーザーID
  $delivery_date: date
) {
  inventory_cart (
    where: {
      user_id: {_eq: $user_id},
      delivery_date: {_eq: $delivery_date}
    },
    order_by: {
      delivery_date: desc
    }
  ) {
    id # item-cart table pk
    user_id
    inventory {
      id
      item {
        id # 商品ID
        name # 商品名
        images # 画像URL 
        item_favorites {
          user_id # このアイテムが好きなuser_id
        }
        characteristic {
          id
          name
          type
        }
        producer {
          name
          email
          tel
          address
        }
        tax_rate
      }
      unit_price # 単価
      ship_date
      available_box_count
    }
    amount # 商品数
    delivery_date # 配達日
  }
}
```

### Response

```graphql
{
  data: {
    inventory_cart: [
      {
        id: Int # item-cart table pk
        user_id: Int # who orderd items
        inventory: {
          id: Int # item-cart table pk
        user_id: Int
        inventory: {
          id: Int
          item: {
            id: Int # 商品ID
            name: String # 商品名
            images: [String] # 画像URL
            item_favorites: {
              user_id: Int # このアイテムが好きなuser_id
            }
            characteristic: {
              id: Int
              name: String
              type: String
            }
            producer: {
              name: String
              email: String
              tel: String
              address: jsonb
            }
            tax_rate: Float
          }
          unit_price: Int # 単価
          ship_date: date
          available_box_count: Int
        }
        amount: Int # 商品数
        delivery_date: date # カートに商品を追加した日時
      }
    ]
  }
}
```

# ピックアップ

## `add_pickup($title, $images, $start_at, $btn_url, $btn_txt, $item_ids)`

【管理画面】ピックアップ登録

### Data Type

item_pickup_insert_input

```graphql
[
  {
    item_id: Int! # 対象商品ID
  }
]
```

### Query

```graphql
mutation add_pickup(
  $title: String! # タイトル
  $summary: String! # 概要
  $images: jsonb # メイン画像URL
  $start_at: date! # 表示開始日時
  $btn_url: String! # ボタン押下時の遷移先URL
  $btn_txt: String! # ボタンのテキスト
  $item_ids: [item_pickup_insert_input!]! # 対象商品リスト
) {
  insert_pickups_one(
    object: {
      title: $title # タイトル
      summary: $summary # 概要
      images: $images # メイン画像URL
      start_at: $start_at # 表示開始日時
      btn_url: $btn_url # ボタン押下時の遷移先URL
      btn_txt: $btn_txt # ボタンのテキスト
      item_pickups: {
        data: $item_ids # 対象商品リスト
      }
    }
  ) {
    id # pickups table pk
    title # タイトル
    summary # 概要
    images # メイン画像URL
    start_at # 表示開始日時
    btn_url # ボタン押下時の遷移先URL
    btn_txt # ボタンのテキスト
    item_pickups {
      item_id # 商品ID
    }
  }
}
```

### Response

```graphql
{
  data: {
    insert_pickups_one: {
      id: Int # pickups table pk
      title: String # タイトル
      summary: String # 概要
      images: String # メイン画像URL
      start_at: date # 表示開始日時
      btn_url: String # ボタン押下時の遷移先URL
      btn_txt: String # ボタンのテキスト
      item_pickups: [
        {
          item_id: Int # 対象商品ID
        }
      ]
    }
  }
}
```

## `get_pickup($pickup_id)`

【アプリ・管理画面用】ID によるピックアップ取得

### Query

```graphql
get_pickup (
  $pickup_id: Int
) {
  pickups (
    where: {
      id: {_eq: $pickup_id}
    }
  ) {
    id # pickups table pk
    title # タイトル
    summary # 概要
    images # メイン画像URL
    start_at # 表示開始日時
    btn_url # ボタン押下時の遷移先URL
    btn_txt # ボタンのテキスト
    item_pickups {
      item {
        id # 商品ID
        name # 商品名
        images # 画像URL
        sales_unit # unit
        inventories {
          id
          unit_price # 単価
        }
        characteristic {
          id
          name
          type
        }
      }
    }
  }
}
```

### Response

```graphql
{
  data: {
    pickups: [
      {
        id: Int # pickups table pk
        title: String # タイトル
        summary: String # 概要
        images: jsonb # メイン画像URL
        start_at: date # 表示開始日時
        btn_url: String # ボタン押下時の遷移先URL
        btn_txt: String # ボタンのテキスト
        item_pickups: [
          {
            item: {
              id: Int # 商品ID
              name: String # 商品名
              images: jsonb # 画像URL
              sales_unit: String # unit
              inventories: {
                id: Int
                unit_price: Int # 単価
              }
              characteristic: {
                id: Int
                name: String
                type: String
              }
            }
          }
        ]
      }
    ]
  }
}
```

## `get_current_pickup($current_date)`

【アプリ・管理画面用】該当日時が到来しているピックアップのうち、最も新しいものを取得

### Query

```graphql
get_current_pickup (
  $current_date: date! #基準日時
) {
  pickups (
    where: {
      start_at: { _lte: $current_date }
    }
  ) {
    id # pickups table pk
    title # タイトル
    images # メイン画像URL
    start_at # 表示開始日時
    btn_url # ボタン押下時の遷移先URL
    btn_txt # ボタンのテキスト
    item_pickups {
      item {
        id # 商品ID
        name # 商品名
        images # 画像URL
        sales_unit # unit
        inventories {
          id
          unit_price # 単価
        }
        characteristic {
          id
          name
          type
        }
      }
    }
  }
}
```

### Response

```graphql
{
  data: {
    pickups: [
      {
        id: Int # pickups table pk
        title: String # タイトル
        summary: String # 概要
        images: jsonb # メイン画像URL
        start_at: date # 表示開始日時
        btn_url: String # ボタン押下時の遷移先URL
        btn_txt: String # ボタンのテキスト
        item_pickups: [
          {
            item: {
              id: Int # 商品ID
              name: String # 商品名
              images: jsonb # 画像URL
              sales_unit: String # unit
              inventories: {
                id: Int
                unit_price: Int # 単価
              }
              characteristic: {
                id: Int
                name: String
                type: String
              }
            }
          }
        ]
      }
    ]
  }
}
```

## `update_pickup($pickup_id, $title, $summary, $images, $start_at, $btn_url, $btn_txt, $item_pickups)`

【管理画面】ピックアップ更新

### Data Type

item_pickup_insert_input

```graphql
[
  {
    item_id: Int! # 商品ID
    pickup_id: Int! # ピックアップID
  }
]
```

### Query

```graphql
mutation update_pickup (
  $pickup_id: Int! # pickups table pk
  $title: String! # タイトル
  $summary: String! # 概要
  $images: jsonb # メイン画像URL
  $start_at: date! # 表示開始日時
  $btn_url: String! # ボタン押下時の遷移先URL
  $btn_txt: String! # ボタンのテキスト
  $item_ids: [item_pickup_insert_input!]! # 対象商品リスト
) {
  delete_item_pickup (
    where: {
      pickup_id: { _eq: $pickup_id } # pickups table pk
    }
  ) {
    affected_rows
  }
  insert_item_pickup {
    objects: $item_pickups # 対象商品リスト
  } {
    affected_rows
  }
  update_pickups_by_pk (
    pk_columns: {
      id: $pickup_id # pickups table pk
    }
    _set: {
      title: $title # タイトル
      summary: $summary # 概要
      images: $images # メイン画像URL
      start_at: $start_at # 表示開始日時
      btn_url: $btn_url # ボタン押下時の遷移先URL
      btn_txt: $btn_txt # ボタンのテキスト
    }
  ) {
      id # pickups table pk
      title # タイトル
      summary # 概要
      images # メイン画像URL
      start_at # 表示開始日時
      btn_url # ボタン押下時の遷移先URL
      btn_txt # ボタンのテキスト
      item_pickups {
        item_id # 商品ID
      }
  }
}
```

### Response

```graphql
{
  data: {
    delete_item_pickup: {
      affected_rows: Int
    },
    insert_item_pickup: {
      affected_rows: Int
    },
    update_pickups_by_pk: {
      id: Int # pickups table pk
      title: String # タイトル
      summary: String # 概要
      images: jsonb # メイン画像URL
      start_at: date # 表示開始日時
      btn_url: String # ボタン押下時の遷移先URL
      btn_txt: String # ボタンのテキスト
      item_pickups: [
        {
          item_id: Int # 商品ID
        }
      ]
    }
  }
}
```

## `remove_pickup($pickup_id)`

【管理画面】ピックアップ削除

### Query

```graphql
mutation remove_pickup(
  $pickup_id: Int! # ピックアップID
) {
  delete_item_pickup (
    where: {
      pickup_id: {_eq: $id}
    }
  ) {
    affected_rows
  }
  delete_pickups (
    where: {
      id: {_eq: $pickup_id}
    }
  ) {
    affected_rows
  }
}
```

### Response

```graphql
{
  data: {
    delete_item_pickup: {
      affected_rows: Int
    },
    delete_pickups: {
      affected_rows: Int
    }
  }
}
```

# Top

## `add_top_content($images, $text, $action, $item_id, $pickup_id, $url)`

【管理画面】トップコンテンツ追加

### Query

```graphql
mutation add_top_content(
  $images: jsonb # 画像URL
  $text: String # テキスト
  $action: Int! # 0: native view - item, 1: native view - pickup, 2: web view
  $item_id: Int # action=0（Nativeページの商品詳細ページを表示）の場合の遷移先商品ID
  $pickup_id: Int # action=1（Nativeページのピックアップページを表示）の場合の遷移先ピックアップID
  $url: String # action=2（WebViewのURLを開く）の場合の遷移先URL
) {
  insert_top_contents_one(
    object: {
      images: $images # 画像URL
      text: $text # テキスト
      action: $action # 0: native view - item, 1: native view - pickup, 2: web view
      item_id: $item_id # 商品ID
      pickup_id: $pickup_id # action=1（Nativeページのピックアップページを表示）の場合の遷移先ピックアップID
      url: $url # action=2（WebViewのURLを開く）の場合の遷移先URL
    }
  ) {
    id # top_contents table pk
    images # 画像URL
    text # テキスト
    action # 0: native view - item, 1: native view - pickup, 2: web view
    item {
      id # 商品ID
      name # 商品名
    }
    pickup {
      id # ピックアップID
      title # ピックアップタイトル
    }
    url # action=2（WebViewのURLを開く）の場合の遷移先URL
  }
}
```

### Response

```graphql
{
  data: {
    insert_top_contents_one: {
      id: Int, # top_contents table pk
      images: jsonb, # 画像URL
      text: String, # テキスト
      action: Int, # 0: native view - item, 1: native view - pickup, 2: web view
      item: {
        id: Int, # 商品ID
        name: String # 商品名
      },
      pickup: {
        id: Int, # ピックアップID
        title: String # ピックアップタイトル
      }
      url: String, # action=2（WebViewのURLを開く）の場合の遷移先URL
    }
  }
}
```

## `get_top_content($content_id)`

【アプリ・管理画面用】ID によるトップコンテンツ取得

### Query

```graphql
get_top_content (
  $content_id: Int # top_contents table pk
) {
  top_contents (
    where: {
      id: {_eq: $content_id}
    }
  ) {
    id # top_contents table pk
    images # 画像URL
    text # テキスト
    action # 0: native view - item, 1: native view - pickup, 2: web view
    item {
      id # action=0（Nativeページの商品詳細ページを表示）の場合の遷移先商品ID
      name # 商品名
    }
    pickup {
      id # action=1（Nativeページのピックアップページを表示）の場合の遷移先ピックアップID
      title # ピックアップタイトル
    }
    url # action=2（WebViewのURLを開く）の場合の遷移先URL
  }
}
```

### Response

```graphql
{
  data: {
    top_contents: [
      {
        id: Int, # top_contents table pk
        images: jsonb, # 画像URL
        text: String, # テキスト
        action: Int, #0: native view - item, 1: native view - pickup, 2: web view
        item: {
          id: Int, # action=0（Nativeページの商品詳細ページを表示）の場合の遷移先商品ID
          name: String # name of item
        },
        pickup: {
          id: Int, # action=1（Nativeページのピックアップページを表示）の場合の遷移先ピックアップID
          title: String # ピックアップタイトル
        }
        url: String, # action=2（WebViewのURLを開く）の場合の遷移先URL
      }
    ]
  }
}
```

## `get_all_top_contents($order_by)`

【アプリ用】トップコンテンツ一覧取得

### Data type

list_order_by

```graphql
{
  data_field: order
}
# example
# { id: asc, name: desc }
```

### Query

```graphql
get_all_top_contents (
  $order_by: list_order_by!
) {
  top_contents (
    order_by: $order_by
  ) {
    id
    images
    text
    action
    item {
      id
      name
    }
    pickup {
      id
      title
    }
    url
  }
}
```

### Response

```graphql
{
  data: {
    top_contents: [
      {
        id: Int,
        images: jsonb,
        text: String,
        action: Int,
        item: {
          id: Int,
          name: String
        },
        pickup: {
          id: Int,
          title: String
        },
        url: String,
      }
    ]
  }
}
```

## `update_top_content($content_id, $images, $text, $action, $item_id, $pickup_id, $url)`

【管理画面用】トップコンテンツ更新

### Query

```graphql
mutation update_top_content(
  $content_id: Int! # top_contents table pk
  $images: jsonb # 画像URL
  $text: String # テキスト
  $action: Int! # 0: native view - item, 1: native view - pickup, 2: web view
  $item_id: Int # 商品ID
  $pickup_id: Int # ピックアップID
  $url: String! # action=2（WebViewのURLを開く）の場合の遷移先URL
) {
  update_top_contents_by_pk (
    pk_columns: {
      id: $content_id
    }
    _set: {
      images: $images # 画像URL
      text: $text # テキスト
      action: $action # 0: native view - item, 1: native view - pickup, 2: web view
      item_id: $item_id # action=0（Nativeページの商品詳細ページを表示）の場合の遷移先商品ID
      pickup_id: $pickup_id # action=1（Nativeページのピックアップページを表示）の場合の遷移先ピックアップID
      url: $url # action=2（WebViewのURLを開く）の場合の遷移先URL
    }
  ) {
    id # top_contents table pk
    images # 画像URL
    text # テキスト
    action # 0: native view - item, 1: native view - pickup, 2: web view
    item {
      id # 商品ID
      name # 商品名
    }
    pickup {
      id # ピックアップID
      title # ピックアップタイトル
    }
    url # action=2（WebViewのURLを開く）の場合の遷移先URL
  }
}
```

### Response

```graphql
{
  data: {
    update_top_contents_by_pk: {
      id: Int, # top_contents table pk
      images: jsonb, # 画像URL
      text: String, # テキスト
      action: Int, # 0: native view - item, 1: native view - pickup, 2: web view
      item: {
        id: Int, # 商品ID
        name: String # 商品名
      },
      pickup: {
        id: Int, # ピックアップID
        title: String # ピックアップタイトル
      }
      url: String, # action=2（WebViewのURLを開く）の場合の遷移先URL
    }
  }
}
```

## `remove_top_content($content_id)`

【管理画面用】トップコンテンツ削除

### Query

```graphql
mutation remove_top_content(
  $content_id: Int # top_contents table pk
) {
  delete_top_contents (
    where: {
      id: {_eq: $content_id}
    }
  ) {
    affected_rows
  }
}
```

### Response

```graphql
{
  data: {
    delete_top_contents: {
      affected_rows: Int
    }
  }
}
```

# お問い合わせ

## `send_email($send_by, $subject, $body, $status, $images, $tel)`

【アプリ用】お問い合わせメール送信

### Query

```graphql
mutation send_email(
  $send_by: String! # メールアドレス
  $subject: String! # お名前
  $body: String! # 問い合わせ内容
  $status: Boolean!   # send status (default: false)
  $images: Jsonb! # s3 bucket image path array
  $tel: String! # phone number
) {
  insert_contacts_one (
    object: {
      subject: $subject
      body: $body
      status: $status
      send_by: $send_by
      images: $images
      tel: $tel
    }
  ) {
    id
    status
  }
}
```

### Response

```graphql
{
  data: {
    insert_contacts_one: {
      id: Int
      status: Boolean
    }
  }
}
```

## `update_contact($contact_id, $status)`

Update contact status.

### Query

```graphql
mutation update_contact (
  $contact_id: Int!
  $status: Boolean!
) {
  update_contacts_by_pk (
    pk_columns: {
      id: $contact_id
    }
    _set: {
      status: $status
    }
  ) {
      id
      status
    }
}
```

### Response

```graphql
{
  data: {
    update_contacts_by_pk: {
      id: Int,
      status: Boolean
    }
  }
}
```

## `get_characteristic_list($characteristic_id)`

Get list of characteristics

### Query

```graphql
get_characteristic_list (
  $characteristic_id: Int 
) {
  characteristics (
    where: {
      id: { _eq: $characteristic_id }
    }
  ) {
      id
      name
      type
    }
}
```

### Response

```graphql
{
  data: {
    characteristics: {
      id: Int,
      name: String,
      type: String
    }
  }
}
```

## `get_buy_history($buyer_shop_id, $date_from, $date_to)`

Get buy history of a current user. (buyer_shop_id stands for org_id in users table)

### Query

```graphql
get_buy_history (
  $buyer_shop_id: Int!
  $date_from: timestamptz
  $date_to: timestamptz
) {
  sales (
    where: {
      buyer_shop_id: { _eq: $buyer_shop_id },
      sales_accept_date: { 
        _gt: $date_from,
        _lt: $date_to
      }
    }
  ) {
    id
    inventory {
      id
      item {
        id # 商品ID
        name # 商品名
        images # 画像URL
        producer {
          name # 生産者名
        }    
        item_favorites {
          user_id # このアイテムが好きなuser_id
        }
        characteristic {
          id
          name
          type
        }
        tax_rate
      }
      unit_price # 単価
      ship_date
      available_box_count
    }
    sales_unit_count
    sales_unit_price
    status
    sales_accept_date
    is_accepted_by_buyer
    invoice_uuid
    delivered_by
    delivered_at
  }
}
```

### Response

```graphql
{
  data: {
    sales: [
      {
        id: Int
        inventory: {
          id: Int
          item: {
            id: Int # 商品ID
            name: String # 商品名
            images: [String] # 画像URL
            producer {
              name: String # 生産者名
            }    
            item_favorites: {
              user_id: Int # このアイテムが好きなuser_id
            }
            characteristic: {
              id: Int
              name: String
              type: String
            }
            tax_rate: Float
          }
          unit_price: Int # 単価
          ship_date: date
          available_box_count: Int
        }
        sales_unit_count: Int
        sales_unit_price: Int
        status: String
        sales_accept_date: date
        is_accepted_by_buyer: Boolean
        invoice_uuid: String
        delivered_by: Int
        delivered_at: date
      }
    ]
  }
}
```

## `get_invoice($invoice_uuid)`

Get invoice according to `invoide_uuid`

### Query

```graphql
get_invoice (
  $invoice_uuid: String 
) {
  invoices (
    where: {
      uuid: { _eq: $invoice_uuid }
    }
  ) {
    id
    buyer_shop_id
    payment_condition
    status
    confirmed_at
    sent_at
    paid_by
    paid_at
    pdf_base64
    invoice_data
  }
}
```

### Response

```graphql
{
  data: {
    invoices: {
      id: Int
      buyer_shop_id: Int
      payment_condition: jsonb
      status: String
      confirmed_at: date
      sent_at: date
      paid_by: Int
      paid_at: date
      pdf_base64: String
      invoice_data: jsonb
    }
  }
}
```

