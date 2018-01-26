# MinMeeting API仕様（β）
MinMeetingはいくつかのWeb APIをβ公開しています。
このAPIを用いると、例えば下記のようなことができます。

- オンラインスケジュールと連携して、会議の10分前にミーティング参加URLを自動的に関係者に送信するようなbotを作成する。
- グループチャットアプリと連携して、「会議をはじめるよ」とつぶやくと、自動的にミーティング参加URLが送られてくるようなbotを作成する。

現状、このAPI仕様はβ公開であり、開発を進めていく上では、仕様変更をせざるを得ない場合が出てくるかもしれませんので、ご了承ください。

## APIトークンの発行
1. googleでログイン<BR>
https://x.minmeeting.com
1. 左メニュー＞設定＞API発行

- 匿名認証ではAPIの発行はできません。

## APIのベースURLと認証方式
### ベースURL
https://x.minmeeting.com/api
### 認証方式
httpのリクエストヘッダーに下記のキーでAPIトークンをセットしてください。
```
X-Minmeeting-API-Token: APIトークン
```

## meeting作成と一時URLの取得
#### リクエスト
| メソッド | パス |
|---|---|
| POST | /meetings |

#### サンプルリクエスト
```
curl -X POST -H "Content-Type:application/json" -H "X-Minmeeting-API-Token: [your api token]" https://x.minmeeting.com/api/meetings
```

#### レスポンス
```.json
{"url":"https://x.minmeeting.com/meetings/xxxxxx/token/xxxxxx"}
```

## 一時URLの再発行
ミーティング参加のための一時URLは、ミーティングの参加者が発行することができますが、一定時間以内に誰も参加しないと画面上からは再度URLを発行することができません。そこで、API経由で一時URLを再発行するAPIを設けました。

#### リクエスト
| メソッド | パス |
|---|---|
| PUT | /meetings/:meetingId |

#### サンプルリクエスト
```
curl -X PUT -H "Content-Type:application/json" -H "X-Minmeeting-API-Token: [your api token]" https://x.minmeeting.com/api/meetings/yourMeetingId
```

#### レスポンス
```.json
{"url":"https://x.minmeeting.com/meetings/xxxxxx/token/xxxxxx"}
```
