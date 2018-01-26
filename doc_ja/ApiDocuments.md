# minmeeting API仕様（β）
minmeetingはいくつかのWeb APIをβ公開しています。
このAPIを用いると、例えば下記のようなことができます。

- オンラインスケジュールと連携して、会議の10分前にミーティング参加URLを自動的に関係者に送信するようなbotを作成する。
- グループチャットアプリと連携して、「会議をはじめるよ」とつぶやくと、自動的にミーティング参加URLが送られてくるようなbotを作成する。

現状、このAPI仕様はβ公開であり、開発を進めていく上では、仕様変更をせざるを得ない場合が出てくるかもしれませんので、ご了承ください。

<BR><BR><BR>
# 1. APIの共通事項

## APIトークンの発行
1. googleでログイン<BR>
https://x.minmeeting.com
1. 左メニュー＞設定＞API発行

- 匿名認証ではAPIの発行はできません。

## APIのベースURLと認証方式（共通ヘッダー）
### ベースURL
https://x.minmeeting.com/api

### 認証方式（共通ヘッダー）
リクエストヘッダーに下記をセットしてください。

| Key | Value |
|---|---|
| Content-Type | application/json |
| X-Minmeeting-API-Token | APIトークン |

## API一覧

| 名称 | パス | 対応種類 |
|---|---|---|
| meeting作成 | /meetings | POST, PUT |
| タイムラインのメッセージ | /meetings/:meetingId/messages | Webhook, POST |
| アジェンダ | /meetings/:meetingId/agendas | Webhook |
| カード | /meetings/:meetingId/agendas/:agendaId/cards | Webhook |

<BR><BR><BR>
# 2. REST API

## meeting作成と一時URLの取得
### Request
#### Method/Path

| Method | Path |
|---|---|
| POST | /meetings |

#### Sample
```
curl -X POST -H "Content-Type:application/json" -H "X-Minmeeting-API-Token: [your api token]" https://x.minmeeting.com/api/meetings
```

### Response
```.json
{"url":"https://x.minmeeting.com/meetings/xxxxxx/token/xxxxxx"}
```

## 一時URLの再発行
ミーティング参加のための一時URLは、ミーティングの参加者が発行することができますが、一定時間以内に誰も参加しないと画面上からは再度URLを発行することができません。そこで、API経由で一時URLを再発行するAPIを設けました。

### Request
#### Method/Path

| Method | Path |
|---|---|
| PUT | /meetings/:meetingId |

#### Sample
```
curl -X PUT -H "Content-Type:application/json" -H "X-Minmeeting-API-Token: [your api token]" https://x.minmeeting.com/api/meetings/yourMeetingId
```

### Response
```.json
{"url":"https://x.minmeeting.com/meetings/xxxxxx/token/xxxxxx"}
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
| text | メッセージ本文。最大1000文字。 | string | ○ |
| author | 作成者表示名 | string |  |

#### Sample
```
curl -X PUT -H "Content-Type:application/json" -H "X-Minmeeting-API-Token: [your api token]" https://x.minmeeting.com/api/meetings/yourMeetingId/messages
```

### Response
```.json
{"messageId": "メッセージID"}
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


## アジェンダのWebhook
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

## カードのWebhook
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
