# Voice Recorder セットアップ手順（Cloudflare R2）

## 1. Cloudflare ダッシュボードで R2 バケットを作成

1. https://dash.cloudflare.com にログイン
2. 左メニューの **R2 Object Storage** をクリック
3. **Create bucket** をクリック
4. バケット名を入力（例: `voice-recordings`）
5. ロケーションは自動（Automatic）でOK → **Create bucket**

## 2. CORS ポリシーを設定

1. 作成したバケットの **Settings** タブを開く
2. **CORS Policy** セクションで **Edit CORS policy** をクリック
3. 以下を貼り付けて保存：

```json
[
  {
    "AllowedOrigins": ["*"],
    "AllowedMethods": ["PUT", "POST"],
    "AllowedHeaders": ["*"],
    "ExposeHeaders": ["ETag"]
  }
]
```

## 3. R2 API トークンを作成

1. R2 の概要ページ右側にある **Manage R2 API Tokens** をクリック
2. **Create API token** をクリック
3. 設定：
   - **Token name**: `voice-recorder-upload`
   - **Permissions**: **Object Read & Write**（一覧取得・再生・ダウンロードにも必要）
   - **Specify bucket(s)**: 手順1で作ったバケットのみに限定
   - **TTL**: 任意（無期限でもOK、個人利用なので）
4. **Create API Token** をクリック
5. 表示される **Access Key ID** と **Secret Access Key** をメモ（Secret は一度しか表示されない）

## 4. Account ID を確認

ダッシュボードの URL に含まれています：
`https://dash.cloudflare.com/<ACCOUNT_ID>/r2/...`

または、R2 の概要ページ右側に表示されています。

## 5. サイトを開いて設定

1. `index.html` をブラウザで開く
2. 右上の「⚙ 設定」をクリック
3. 以下を入力：
   - **R2 Bucket Name**: `voice-recordings`（手順1のバケット名）
   - **Cloudflare Account ID**: 手順4で確認したもの
   - **R2 Access Key ID**: 手順3で発行したもの
   - **R2 Secret Access Key**: 手順3で発行したもの
   - **保存先フォルダ**: `recordings/`（デフォルトのままでOK）
4. 「保存」をクリック

## 6. 使い方

赤い丸ボタンを押すと録音開始。もう一度押すと停止＆自動アップロード。

ファイルは `recordings/YYYY-MM-DD/HHmmss.webm` の形式で保存されます。

## コスト目安

Cloudflare R2 の料金（2026年3月時点）:

| 項目 | 無料枠 | 超過分 |
|------|--------|--------|
| ストレージ | 10 GB/月 | $0.015/GB/月 |
| Class A 操作（PUT等） | 100万回/月 | $4.50/100万回 |
| Class B 操作（GET等） | 1000万回/月 | $0.36/100万回 |
| エグレス（外部送信） | **無料** | **無料** |

個人利用なら **無料枠で十分** 収まります（1日10分録音 × 30日 = 約18MB）。
S3 と比べてエグレス料金が完全無料なのが最大のメリットです。
