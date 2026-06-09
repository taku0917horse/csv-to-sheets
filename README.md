# CSV → Google スプレッドシート自動転記ツール

CSVファイルの内容を、ブラウザだけで Google スプレッドシートに転記できるシングルページツールです。  
サーバー不要・インストール不要。HTMLファイルを開くだけで動作します。

---

## 概要

| 項目 | 内容 |
|------|------|
| 動作環境 | モダンブラウザ（Chrome / Edge / Firefox 推奨） |
| 依存ライブラリ | Google Identity Services（CDN 読み込み） |
| 使用 API | Google Sheets API v4 |
| 認証方式 | OAuth 2.0（インポジット型トークンフロー） |
| サーバー | 不要（完全クライアントサイド） |

### 主な機能

- CSV ファイルのドラッグ＆ドロップ / クリック選択
- 文字コード自動選択（UTF-8 / Shift-JIS / EUC-JP）
- 読み込みデータのプレビュー表示（行数・列数をカウント）
- Google アカウントで認証後、指定スプレッドシートへ一括書き込み
- 書き込み先のシート名・開始セルを自由に指定

---

## 使い方

### 1. HTMLファイルを開く

`csv-to-sheets.html` をブラウザで開きます。  
VS Code の Live Server 等を使うと `http://localhost:5500` のような URL で開けます。

### 2. 設定欄を入力する

| 設定項目 | 説明 |
|----------|------|
| **OAuth クライアント ID** | Google Cloud Console で取得した OAuth 2.0 クライアント ID（必須） |
| **API キー** | 省略可。割り当て管理目的で任意設定 |
| **スプレッドシート ID** | 転記先スプレッドシートの ID（必須） |
| **書き込み範囲** | 書き込みを開始するセル（例: `Sheet1!A1`、`シート1!B2`） |

**スプレッドシート ID の調べ方**

```
https://docs.google.com/spreadsheets/d/【ここがID】/edit
```

### 3. Google アカウントで認証する

「Google アカウントで認証」ボタンをクリックし、ポップアップで Google アカウントにログインします。  
認証に成功するとバッジが「認証済み ✓」に変わります。

> アクセストークンの有効期限は約1時間です。期限切れになった場合は再認証してください。

### 4. CSV ファイルを読み込む

ドロップゾーンに `.csv` ファイルをドラッグ＆ドロップするか、クリックしてファイルを選択します。  
読み込み後、データのプレビューが表示されます。

### 5. スプレッドシートに転記する

「スプレッドシートに転記」ボタンをクリックします。  
成功すると「転記完了！ N セルを更新しました」と通知が表示されます。

---

## Google OAuth 2.0 の設定方法

### 手順

1. **Google Cloud Console にアクセス**  
   [https://console.cloud.google.com/](https://console.cloud.google.com/) を開き、プロジェクトを作成または選択します。

2. **Google Sheets API を有効化**  
   左メニュー「APIとサービス」→「ライブラリ」→「Google Sheets API」を検索し、「有効にする」をクリックします。

3. **OAuth 同意画面を設定**  
   「APIとサービス」→「OAuth 同意画面」を開き、以下を設定します。
   - ユーザーの種類: **外部**
   - アプリ名・メールアドレスを入力
   - スコープに `Google Sheets API` → `.../auth/spreadsheets` を追加

4. **OAuth 2.0 クライアント ID を作成**  
   「APIとサービス」→「認証情報」→「認証情報を作成」→「OAuth クライアント ID」を選択します。
   - アプリの種類: **ウェブアプリケーション**
   - 「承認済みの JavaScript 生成元」に、このHTMLを開いているオリジンを追加します。

   | 開き方 | 登録するオリジン |
   |--------|-----------------|
   | Live Server (VS Code) | `http://localhost:5500` |
   | ローカルファイルを直接開く | `http://localhost` または `null`（非推奨）|
   | 独自ドメインで公開 | `https://example.com` |

5. **クライアント ID をコピーして貼り付け**  
   作成後に表示される `xxxx.apps.googleusercontent.com` 形式の文字列を、ツールの「OAuth クライアント ID」欄に貼り付けます。

---

## 必要な API

### Google Sheets API v4

CSVデータをスプレッドシートに書き込むために使用します。

- **エンドポイント**: `PUT https://sheets.googleapis.com/v4/spreadsheets/{spreadsheetId}/values/{range}`
- **パラメータ**: `valueInputOption=RAW`（値をそのまま文字列として書き込む）
- **認証**: OAuth 2.0 Bearer トークン（`Authorization: Bearer <access_token>`）
- **スコープ**: `https://www.googleapis.com/auth/spreadsheets`

### Google Identity Services (GIS)

OAuth 2.0 の認証フロー（トークンリクエスト）を処理するための Google 公式ライブラリです。

- **読み込み元**: `https://accounts.google.com/gsi/client`（CDN）
- **フロー**: Implicit Flow（アクセストークンをブラウザ内で直接取得）

> API キーは必須ではありません。設定することでリクエストの割り当て管理や Cloud Console でのトラブルシューティングが容易になります。

---

## ライセンス

MIT
