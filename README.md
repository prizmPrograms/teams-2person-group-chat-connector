# Create 2-person group chat (Microsoft Teams) — Power Automate カスタムコネクタ

Microsoft Teams で **2 人のグループチャット**を作成する Power Automate / Power Apps 用のカスタムコネクタです。Microsoft Graph の `POST /v1.0/chats` を呼び出します。

通常の 1 対 1 チャットと違い、グループチャットなら**表示名（トピック）を自由に設定**でき、既存の個人チャットと分離できます。Power Automate の標準アクションでは `/v1.0/chats` を直接呼べないため、カスタムコネクタで実現しています。

## できること

- アクション **「2人のグループチャットを作成する」（operationId: `CreateGroupChat`）**
- 入力は **グループ名（topic）** と **ユーザー 2 名（自分を含む）** だけ
  - `chatType`（`group`）、`@odata.type`、`roles`（`owner`）などの固定値は内部設定（`x-ms-visibility: internal`）で隠してあります

## このリポジトリのファイル

| ファイル | 内容 |
| -- | -- |
| `apiDefinition.swagger.json` | コネクタの OpenAPI (Swagger 2.0) 定義。**機密情報は含まれていません。** |
| `apiProperties.json` | 認証（OAuth 2.0 / Microsoft Entra）の設定。`clientId`・`tenantId` は**プレースホルダー**です。 |
| `.gitignore` | `settings.json`（paconn のローカル設定。機密が入りうる）などを除外 |
| `LICENSE` | MIT |

> ⚠️ **クライアントシークレットはこのリポジトリには含まれていません**（含めないでください）。インポート後に各自の環境で設定します。

## 前提

- **有償ライセンス**：カスタム/プレミアムコネクタを含むフローの実行には、プレミアム対応ライセンス（Power Automate Premium など）が必要です。評価用には 90 日間の試用版も使えます。
- **Microsoft Entra のアプリ登録**（委任権限 `Chat.Create`、必要に応じて `User.Read`）と**クライアントシークレット**。

## セットアップ手順

### 1. Microsoft Entra でアプリを登録

1. [Microsoft Entra 管理センター](https://entra.microsoft.com/) → `Entra ID` → `アプリの登録` → `新規登録`。
2. 登録後、`アプリケーション (クライアント) ID` と `ディレクトリ (テナント) ID` を控える。
3. `証明書とシークレット` → `新しいクライアント シークレット` を作成し、表示された**「値」**を控える（「シークレット ID」ではありません。一度しか表示されません）。
4. `API のアクセス許可` → `Microsoft Graph` → `委任されたアクセス許可` → `Chat.Create`（＋必要なら `User.Read`）を追加。

### 2. コネクタをインポート

**方法 A: メーカーポータル（GUI）**

1. [Power Automate](https://make.powerautomate.com/) → `カスタム コネクタ` → `カスタム コネクタの新規作成` → `OpenAPI ファイルをインポートします` で `apiDefinition.swagger.json` を選択。
2. `セキュリティ` タブで OAuth 2.0（Azure Active Directory）を選び、`Client ID`・`Client secret`（手順 1 の値）・`Tenant ID`・`Resource URL`（`https://graph.microsoft.com`）を入力して保存。
3. 保存後に発行される **リダイレクト URL** をコピーし、Entra アプリの `認証` → `Web` のリダイレクト URI に追加。

**方法 B: paconn（Power Platform Connectors CLI）**

```bash
pip install paconn
paconn login
# apiProperties.json の clientId / tenantId を自分の値に書き換えてから:
paconn create -s settings.json   # または apiDefinition / apiProperties を指定して作成
```

> `apiProperties.json` の `<YOUR_ENTRA_APP_CLIENT_ID>` `<YOUR_TENANT_ID_OR_common>` を自分の値に置き換えてください。client secret は paconn 実行時に対話的に入力します（ファイルには保存しないこと）。

### 3. 接続を作成して使う

1. フローに「2人のグループチャットを作成する」アクションを追加し、**接続を作成**（サインイン＆同意）。
2. 入力:
   - **グループ名**：チャットの表示名
   - **ユーザー（2 名）**：`https://graph.microsoft.com/v1.0/users('メールアドレス')` の形式。**自分を含む 2 名**を指定（例: 自分と相手）。

成功すると `201 Created` が返り、グループチャットが作成されます。

## 補足・注意

- このコネクタは委任権限で動くため、接続したユーザー自身がメンバーに含まれている必要があります（Graph の仕様）。
- クライアントシークレットには**有効期限**があります。切れると接続が失敗するので、更新したら接続を再認証してください。
- 参考記事: https://zenn.dev/prizmprograms/articles/0655808e038f98

## License

[MIT](./LICENSE)
