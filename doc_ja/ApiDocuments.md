# minmeeting API仕様（β）
minmeetingはいくつかのWeb APIをβ公開しています。
このAPIを用いると、例えば下記のようなことができます。

- オンラインスケジュールと連携して、会議の10分前にミーティング参加URLを自動的に関係者に送信するようなbotを作成する。
- グループチャットアプリと連携して、「会議をはじめるよ」とつぶやくと、自動的にミーティング参加URLが送られてくるようなbotを作成する。
- minmeetingでの発言に応じて、参考情報を表示したり、問いかけをするファシリテーションbotを作成する。
- グループチャットの対話内容をminmeetingに連携してアジェンダごとに整理し、議事録を作る。
- （ユーザが操作をしたタイミングで）minmeetingの内容を議事録として、外部のノートアプリに連携する。
- （ユーザが操作をしたタイミングで）minmeetingの内容を外部のToDoアプリに連携する。

現状、このAPI仕様はβ公開であり、開発を進めていく上では、仕様変更をせざるを得ない場合が出てくるかもしれませんので、ご了承ください。

<BR><BR><BR>
# 1. APIの共通事項

## APIトークンの発行
1. googleでログイン<BR>
https://ten.minmeeting.com
1. 左メニュー＞設定＞API発行

- 匿名認証ではAPIの発行はできません。
- 2018年4月1日時点で、セキュリティ強化のためAPIトークンの形式が変わりました。大変お手数ですが、設定画面よりAPIトークンを再発行してください。

## APIのベースURLと認証方式（共通ヘッダー）
### ベースURL
https://ten.minmeeting.com/api

### 認証方式（共通ヘッダー）
リクエストヘッダーに下記をセットしてください。

| Key | Value |
|---|---|
| Content-Type | application/json |
| X-Minmeeting-API-Token | APIトークン |

## API一覧

| 名称 | パス | 対応種類 |
|---|---|---|
| ミーティング | /meetings | Webhook（※将来）, GET（※）, POST, PUT |
| アジェンダ | /meetings/:meetingId/agendas | Webhook（※将来）, GET（※将来）, POST, PUT |
| カード | /meetings/:meetingId/agendas/:agendaId/cards | Webhook（※将来）, GET（※将来）, POST, PUT |
| タイムラインのメッセージ | /meetings/:meetingId/messages | Webhook（※）, GET（※）, POST, PUT |

「将来」と書いてあるAPIは現在まだ対応していません。

### 制限事項
無料プランでのご利用の場合、下記の制限があります。
- APIの呼び出し回数は1週間に50回までとします。
- 上記API一覧に「※」マークが付いているAPIは、ご利用いただけません。

<BR><BR><BR>
---

# 2. REST API

## meeting統計とアジェンダ・カード一覧の取得
### Request
#### Method/Path

| Method | Path |
|---|---|
| GET | /meetings/:meetingId |

#### Parameters

| Key | Value | Required |
|---|---|---|
| requestAgendas | true: アジェンダデータを取得する。 false（または指定なし）: アジェンダデータを取得しない。 |  |

#### Sample
```
curl -X GET -H "Content-Type:application/json" -H "X-Minmeeting-API-Token: ${yourApiToken}"  https://ten.minmeeting.com/api/meetings/${yourMeetingId}
```

### Response
```.json
{
  "meetingId": "ミーティングID",
  "title": "ミーティングタイトル",
  "statistics": {
    "durationInMillisec": "会議時間（ミリ秒）",
    "estimatedInMillisec": "会議予定時間（ミリ秒）",
    "messagesCount": "メッセージ件数（発言数）",
    "cardsCount": "カード件数（議事録としてアジェンダ上に整理された件数）",
    "cardReactionsCount": "リアクション件数",
    "startedDate": "会議開始日時"
  },
  "members": {"$userId": "ユーザ名"},
  "agendas": {
    "$agendaId": {
      "title": "アジェンダタイトル",
      "duration": "予定時間（分）",
      "order": "表示順",
      "at": "作成日時（UNIXタイムスタンプ：ミリ秒）",
      "cards": {
        "$cardId": {
          "text": "本文",
          "author": "作成者名",
          "by": "作成者ID",
          "at": "作成時刻（UNIXタイムスタンプ：ミリ秒）"
        }
      }
    }
  }
}
```

## meeting作成と一時URLの取得
### Request
#### Method/Path

| Method | Path |
|---|---|
| POST | /meetings |

#### Body

| Key | SubKey | Value | Type | Required |
|---|---|---|---|---|
| title | - | ミーティングタイトル | string |  |
| agendas | - | アジェンダ一覧 | array |  |
| - | title | アジェンダタイトル。最大100文字。 | string | ○ |
| - | duration | アジェンダの予定時間（分）。最大60分。 | number |  |

- 一時URLを発行したいだけならbodyは省略可。
- アジェンダは登録したい順番に従って配列で記述する。

#### Sample
一時URLのみ発行する場合
```
curl -X POST -H "Content-Type:application/json" -H "X-Minmeeting-API-Token: ${yourApiToken}"  https://ten.minmeeting.com/api/meetings
```

ミーティング作成と同時にアジェンダを登録する場合
```
curl -X POST -H "Content-Type:application/json" -H "X-Minmeeting-API-Token: ${yourApiToken}" -d '{"agendas": [{"title": "アジェンダその1", "duration": 2}, {"title": "アジェンダその2", "duration": 1}]}' https://ten.minmeeting.com/api/meetings
```

### Response
```.json
{
  "meetingId": "yourMeetingId",
  "url":"https://ten.minmeeting.com/meetings/xxxxxx/token/xxxxxx"
}
```

## 一時URLの再発行
ミーティング参加のための一時URLは、ミーティングの参加者が発行することができますが、一定時間以内に誰も参加しないと画面上からは再度URLを発行することができません。そこで、API経由で一時URLを再発行するAPIを設けました。

### Request
#### Method/Path

| Method | Path |
|---|---|
| PUT | /meetings/:meetingId |

#### Body
| Key | SubKey | Value | Type | Required |
|---|---|---|---|---|
| title | - | ミーティングタイトル | string |  |
| agendas | - | アジェンダ一覧 | array |  |
| - | title | アジェンダタイトル。最大100文字。 | string | ○ |
| - | duration | アジェンダの予定時間（分）。最大60分。 | number |  |

- 一時URLを再発行したいだけならbodyは省略可。
- アジェンダは登録したい順番に従って配列で記述する。
- 既存のアジェンダは変更せず、追加をする。

#### Sample
一時URLを再発行したい場合
```
curl -X PUT -H "Content-Type:application/json" -H "X-Minmeeting-API-Token: ${yourApiToken}" https://ten.minmeeting.com/api/meetings/${yourMeetingId}
```

アジェンダを追加したい場合
```
curl -X PUT -H "Content-Type:application/json" -H "X-Minmeeting-API-Token: ${yourApiToken}" -d '{"agendas": [{"title": "アジェンダその1", "duration": 2}, {"title": "アジェンダその2", "duration": 1}]}' https://ten.minmeeting.com/api/meetings/${yourMeetingId}
```

### Response
```.json
{
  "meetingId": "yourMeetingId",
  "url":"https://ten.minmeeting.com/meetings/xxxxxx/token/xxxxxx"
}
```

---

## アジェンダの取得
### Request
#### Method/Path

| Method | Path |
|---|---|
| GET | /meetings/:meetingId/agendas/:agendaId |

#### Parameters
なし

#### Sample
```
curl -X GET -H "Content-Type:application/json" -H "X-Minmeeting-API-Token: ${yourApiToken}" https://ten.minmeeting.com/api/meetings/${yourMeetingId}/agendas/${yourAgendId}
```

### Response
```.json
{
  "meetingId": "ミーティングID",
  "agendaId": "アジェンダID",
  "title": "アジェンダタイトル",
  "duration": "予定時間（分）",
  "order": "表示順",
  "at": "作成日時（UNIXタイムスタンプ：ミリ秒）",
  "cards": {
    "$cardId": {
      "text": "タイトル",
      "author": "作成者名",
      "by": "作成者ID",
      "at": "作成時刻（UNIXタイムスタンプ：ミリ秒）"
    }
  }
}
```

## アジェンダの投稿
### Request
#### Method/Path

| Method | Path |
|---|---|
| POST | /meetings/:meetingId/agendas |

#### Body

| Key | Value | Type | Required |
|---|---|---|---|
| title | アジェンダタイトル。最大100文字。 | string | ○ |
| duration | アジェンダの予定時間（分）。最大60分。 | number |  |

#### Sample
```
curl -X POST -H "Content-Type:application/json" -H "X-Minmeeting-API-Token: ${yourApiToken}" -d '{"title": "アジェンダその1", "duration": 2}' https://ten.minmeeting.com/api/meetings/${yourMeetingId}/agendas
```

### Response
```.json
{
  "meetingId": "ミーティングID",
  "agendaId": "アジェンダID",
  "title": "アジェンダタイトル",
  "duration": "アジェンダの予定時間",
  "agendaNumber": "アジェンダ番号",
  "order": "表示順",
  "at": "作成日時（UNIXタイムスタンプ：ミリ秒）"
}
```

## アジェンダの更新
### Request
#### Method/Path

| Method | Path |
|---|---|
| PUT | /meetings/:meetingId/agendas/:agendaId |

#### Body

| Key | Value | Type | Required |
|---|---|---|---|
| title | アジェンダタイトル。最大100文字。 | string |  |
| duration | アジェンダの予定時間（分）。最大60分。 | number |  |

いずれかのフィールドは必須。

#### Sample
```
curl -X PUT -H "Content-Type:application/json" -H "X-Minmeeting-API-Token: ${yourApiToken}" -d '{"title": "アジェンダその1", "duration": 2}' https://ten.minmeeting.com/api/meetings/${yourMeetingId}/agendas/${agendaIdToUpdate}
```

### Response
```.json
{
  "meetingId": "ミーティングID",
  "agendaId": "アジェンダID"
}
```

---

## カード一覧の取得
### Request
#### Method/Path

| Method | Path |
|---|---|
| GET | /meetings/:meetingId/agendas/:agendaId/cards |

#### Sample
```
curl -X GET -H "Content-Type:application/json" -H "X-Minmeeting-API-Token: ${yourApiToken}" https://ten.minmeeting.com/api/meetings/${yourMeetingId}/agendas/${agendaId}/cards
```

### Response
```.json
{
  "meetingId": "ミーティングID",
  "agendaId": "アジェンダID",
  "cards": [
    {
      "cardId": "カードID",
      "text": "カード本文",
      "author": "作成者",
      "order": "表示順",
      "at": "作成日時（UNIXタイムスタンプ：ミリ秒）"
    }
  ]
}
```

## カードの取得
### Request
#### Method/Path

| Method | Path |
|---|---|
| GET | /meetings/:meetingId/agendas/:agendaId/cards/:cardId |

#### Parameters
なし

#### Sample
```
curl -X GET -H "Content-Type:application/json" -H "X-Minmeeting-API-Token: ${yourApiToken}" https://ten.minmeeting.com/api/meetings/${yourMeetingId}/agendas/${agendaId}/cards/${cardId}
```

### Response
```.json
{
  "meetingId": "ミーティングID",
  "agendaId": "アジェンダID",
  "cardId": "カードID",
  "text": "カード本文",
  "author": "作成者",
  "order": "表示順",
  "at": "作成日時（UNIXタイムスタンプ：ミリ秒）"
}
```

## カードの投稿
### Request
#### Method/Path

| Method | Path |
|---|---|
| POST | /meetings/:meetingId/agendas/:agendaId/cards |

#### Body

| Key | Value | Type | Required |
|---|---|---|---|
| text | カード本文。最大1000文字。 | string |  |
| author | 作成者表示名。最大100文字。 | string |  |

いずれかのフィールドは必須。

#### Sample
```
curl -X POST -H "Content-Type:application/json" -H "X-Minmeeting-API-Token: ${yourApiToken}" -d '{"text": "こんにちは"}' https://ten.minmeeting.com/api/meetings/${yourMeetingId}/agendas/${agendaIdToUpdate}/cards
```

### Response
```.json
{
  "meetingId": "ミーティングID",
  "agendaId": "アジェンダID",
  "cardId": "カードID",
  "text": "カード本文",
  "author": "作成者",
  "order": "表示順",
  "at": "作成日時（UNIXタイムスタンプ：ミリ秒）"
}
```

## カードの更新
### Request
#### Method/Path

| Method | Path |
|---|---|
| PUT | /meetings/:meetingId/agendas/:agendaId/cards/:cardId |

#### Body

| Key | Value | Type | Required |
|---|---|---|---|
| text | メッセージ本文。最大1000文字。 | string |  |
| author | 作成者表示名。最大100文字。 | string |  |

いずれかのフィールドは必須。

#### Sample
```
curl -X PUT -H "Content-Type:application/json" -H "X-Minmeeting-API-Token: ${yourApiToken}" -d '{"text": "こんにちは"}' https://ten.minmeeting.com/api/meetings/${yourMeetingId}/agendas/${agendaIdToUpdate}/cards/${cardIdToUpdate}
```

### Response
```.json
{
  "meetingId": "ミーティングID",
  "agendaId": "アジェンダID",
  "cardId": "カードID"
}
```

---

## メッセージ一覧の取得
### Request
#### Method/Path

| Method | Path |
|---|---|
| GET | /meetings/:meetingId/messages |

#### Parameters

| Key | Value | Required |
|---|---|---|
| olderThan | 前のページの一番古いデータの作成日時（前ページのoldestTimestampを指定する。UNIXタイムスタンプ：ミリ秒） |  |
| limit | 1ページあたりの件数。デフォルトは20件。上限は200件。 |  |

#### Sample
```
curl -X GET -H "Content-Type:application/json" -H "X-Minmeeting-API-Token: ${yourApiToken}"  https://ten.minmeeting.com/api/meetings/${yourMeetingId}/messages
```

### Response
```.json
{
  "meetingId": "ミーティングID",
  "oldestTimestamp": "一番古いデータのタイムスタンプ（UNIXタイムスタンプ：ミリ秒）"
  "messages": [
    {
      "messageId": "メッセージID",
      "text": "メッセージ本文",
      "author": "作成者",
      "at": "作成日時（UNIXタイムスタンプ：ミリ秒）"
    }
  ]
}
```

## メッセージの取得
### Request
#### Method/Path

| Method | Path |
|---|---|
| GET | /meetings/:meetingId/messages/:messageId |

#### Parameters
なし

#### Sample
```
curl -X GET -H "Content-Type:application/json" -H "X-Minmeeting-API-Token: ${yourApiToken}"  https://ten.minmeeting.com/api/meetings/${yourMeetingId}/messages/${messageId}
```

### Response
```.json
{
  "meetingId": "ミーティングID",
  "messageId": "メッセージID",
  "text": "メッセージ本文",
  "author": "作成者",
  "at": "作成日時（UNIXタイムスタンプ：ミリ秒）"
}
```

## メッセージの投稿
### Request
#### Method/Path

| Method | Path |
|---|---|
| POST | /meetings/:meetingId/messages |

#### Body

| Key | Value | Type | Required |
|---|---|---|---|
| text | メッセージ本文。最大1000文字。 | string |  |
| author | 作成者表示名。最大100文字。 | string |  |

いずれかのフィールドは必須。

#### Sample
```
curl -X POST -H "Content-Type:application/json" -H "X-Minmeeting-API-Token: ${yourApiToken}" -d '{"text": "こんにちは"}' https://ten.minmeeting.com/api/meetings/${yourMeetingId}/messages
```

### Response
```.json
{
  "meetingId": "ミーティングID",
  "messageId": "メッセージID",
  "text": "メッセージ本文",
  "author": "作成者",
  "at": "作成日時（UNIXタイムスタンプ：ミリ秒）"
}
```

## メッセージの更新
### Request
#### Method/Path

| Method | Path |
|---|---|
| PUT | /meetings/:meetingId/messages/:messageId |

#### Body

| Key | Value | Type | Required |
|---|---|---|---|
| text | メッセージ本文。最大1000文字。 | string |  |
| author | 作成者表示名。最大100文字。 | string |  |

いずれかのフィールドは必須。

#### Sample
```
curl -X PUT -H "Content-Type:application/json" -H "X-Minmeeting-API-Token: ${yourApiToken}" -d '{"text": "こんにちは"}' https://ten.minmeeting.com/api/meetings/${yourMeetingId}/messages/${messageIdToUpdate}
```

### Response
```.json
{"meetingId": "ミーティングID", "messageId": "メッセージID"}
```

<BR><BR><BR>
# 3. Webhook
minmeetingで発生したイベントを設定画面で指定したURLに通知します。通知はPOSTメソッドで行います。

## Webhookのリクエストの検証方法
Webhookのリクエストがminmeetingから発行された正統なものかを検証するため、Webhookのリクエストヘッダには下記の通り検証用トークンをセットします。
トークンが自分のものと一致することをもって、正統なリクエストと判定してください。
検証用トークンは設定画面にて取得することができます。

| Key | Value |
|---|---|
| Content-Type | application/json |
| X-Minmeeting-Verification-Token | 検証用トークン |

## メッセージのWebhook
タイムラインにメッセージが投稿されたタイミングで、指定したURLにPOSTします。

### Request

| Key | Value | Type |
|---|---|---|
| meetingId | ミーティングID | string |
| messageId | メッセージID | string |
| text | メッセージ本文 | string |
| author | 作成者表示名 | string |

### Response
リクエストに対し、ステータスコード200で下記のようにレスポンスを返すと、メッセージを投稿することもできます。

#### Statu Code

```
200
```

#### Response Header
レスポンスでメッセージ投稿をする場合は下記の通りヘッダーをセットしてください。メッセージ投稿をしない場合は不要です。

| Key | Value |
|---|---|
| Content-Type | application/json |
| X-Minmeeting-API-Token | APIトークン |

#### Body
レスポンスでメッセージ投稿をする場合は下記の通りBodyをセットしてください。メッセージ投稿をしない場合は不要です。

| Key | Value | Type | Required |
|---|---|---|---|
| text | メッセージ本文。最大1000文字。 | string | ○ |
| author | 作成者表示名 | string |  |


## アジェンダのWebhook（将来）
ユーザがアジェンダの出力を選択したタイミングで、指定したURLにPOSTします。

### Request

| Key | Value | Type |
|---|---|---|
| meetingId | ミーティングID | string |
| agendaId | アジェンダID | string |
| title | アジェンダタイトル | string |
| duration | 予定時間（分） | number |

### Response
#### Statu Code

```
200
```

## カードのWebhook（将来）
ユーザがカードの出力を選択したタイミングで、指定したURLにPOSTします。

### Request

| Key | Value | Type |
|---|---|---|
| meetingId | ミーティングID | string |
| agendaId | アジェンダID | string |
| cardId | カードID | string |
| text | カード本文 | string |
| color | カード色 | object |
| reactions | リアクション | object |
| tags | タグ | object |

### Response
#### Statu Code

```
200
```
