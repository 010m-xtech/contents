# サービス向上アンケート（属性収集＆店舗別ルーティング）

ミニゲームやコンテンツへの遷移前にユーザーの属性（性別・年代）を収集する、スマートフォン向けのシンプル＆クリーンなアンケート画面です。  
URLパラメータからの店舗ID判定、端末ID（`device_id`）の自動発行・引き継ぎ、Google Apps Script（GAS）への回答・行動ログ送信、および `routing.json` に基づく店舗別の自動リダイレクト機能を備えています。

## フォルダ構成

```text
├── index.html        … アンケート画面本体（UI・バリデーション・GAS送信・遷移制御）
├── routing.json      … 店舗ごとの遷移先URL・コンテンツ名設定（★ここを編集して遷移先を変更）
├── user-male.svg     … 性別アイコン（男性）
├── user-female.svg   … 性別アイコン（女性）
├── under35.svg       … 年代アイコン（35歳未満）
└── over35.svg        … 年代アイコン（35歳以上）
```

## 主な機能と仕組み

- **URLパラメータからの店舗ID取得 (`getShopId`)**
  - アクセスされたURLのクエリパラメータ `?shop_id=123456` または `?cd=123456` から店舗IDを自動取得します（他ゲームのような `#` ハッシュ形式ではなく `?` パラメータ形式に対応）。

- **端末ID (`device_id`) の自動発行と引き継ぎ**
  - 初回アクセス時に一意の識別ID（例: `dev_xxxxxxxx...`）を生成して端末の `localStorage` に保存します。
  - アンケート送信後、遷移先のURLに `?device_id=dev_xxx` を自動付与してリダイレクトするため、遷移先のゲームやポータル側でも同一ユーザーの行動を追跡できます。

- **GAS（Google Apps Script）への2段階データ送信**
  - **画面表示時 (`survey_view`)**: アンケート画面を開いたタイミングで閲覧イベントログを送信します。
  - **回答送信時 (`survey_submit`)**: 回答された属性データ（`type: 'user'`：性別・年代）と、回答完了イベント（`type: 'event'`：遷移先コンテンツ名）を並行送信したうえで次画面へ遷移します。

- **`routing.json` による店舗別遷移先コントロール**
  - 取得した店舗IDに対応する遷移先URLとコンテンツ名を `routing.json` から読み込みます。
  - 店舗IDの指定がない場合や未登録の店舗IDの場合は、自動的に `default` の設定先へ遷移します。

---

## 設定ファイル（routing.json）の書き方

`index.html` と同じ階層に `routing.json` を配置し、店舗IDごとの遷移先URL（`url`）とログ記録用の名称（`name`）を定義します。

```json
{
  "default": {
    "name": "ゲームポータル（共通）",
    "url": "[https://example.com/portal/#000000](https://example.com/portal/#000000)"
  },
  "123456": {
    "name": "〇〇店 専用ポータル",
    "url": "[https://example.com/portal/#123456](https://example.com/portal/#123456)"
  }
}
