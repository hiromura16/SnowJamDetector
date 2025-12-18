# スノージャム検知システム

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Version](https://img.shields.io/badge/version-0.3.0-green.svg)
![Python](https://img.shields.io/badge/python-3.11-blue.svg)
![Node](https://img.shields.io/badge/node-20+-green.svg)

定点カメラから送信される画像を監視し、スノージャム（雪の詰まり）を自動検知するシステムです。

**現在のバージョン**: 0.3.0

## 機能

- 📸 定点カメラからの画像をFTPで自動受信
- 🔍 画像差分によるスノージャムの自動検知
- 🚨 検知時のアラート（Slack通知、GPIO出力）
- 📊 Web UIによるダッシュボードと設定管理
- 💾 画像の自動保存と90日間のリテンション
- ⏱️ 画像到着遅延の監視

## クイックスタート

### 前提条件

- Docker 20.10以上と Docker Compose 2.0以上がインストールされていること
- ポート80（Web UI）とポート21（FTP）が使用可能であること

### 5分で始める

```bash
# 1. リポジトリをクローン
git clone <repository-url>
cd SnowJamDetector

# 2. 設定ファイルを準備
cp config/settings.json.example config/settings.json
# config/settings.json を編集して、必要に応じてSlack通知などを設定

# 3. Docker Composeで起動
docker compose up --build -d

# 4. ブラウザで http://localhost にアクセス
```

詳細な手順は [デプロイ手順書](docs/deployment.md) を参照してください。

## セットアップ詳細

### 1. 設定ファイルの準備

初回セットアップ時は、テンプレートから設定ファイルをコピーします:

```bash
cp config/settings.json.example config/settings.json
```

`config/settings.json` を編集して、以下の項目を設定してください:

- **検知パラメータ**: `threshold`, `consecutive_hits`, `binary_threshold`, `blur_kernel`
- **遅延監視**: `delay_monitor_enabled`, `delay_threshold_seconds`
- **アラート設定**: `alarm_enabled`, `slack_webhook_url`, `slack_bot_token`, `slack_channel`, `gpio_pin`

設定ファイルの詳細は `config/settings.json.example` を参照してください。

### 2. マスクファイルの設定（オプション）

検知範囲を限定したい場合は、モノクロのPNGまたはJPEG画像を `config/mask.png` として配置してください。
- **白い領域**: 検知対象（`mask_inclusive: true` の場合）
- **黒い領域**: 検知対象外（`mask_inclusive: true` の場合）

マスクファイルがない場合は、画像全体が検知対象になります。
マスクはWeb UIの設定画面からもアップロード・変更できます。

#### マスク画像の作成方法

1. カメラから取得した画像と同じサイズの画像を用意
2. 画像編集ソフト（GIMP、Photoshop、ペイントなど）で編集
3. 検知したい領域を**白**、検知したくない領域を**黒**で塗りつぶす
4. モノクロ（グレースケール）で保存（PNGまたはJPEG形式）
5. `config/mask.png` として保存

リポジトリにはサンプルマスク画像（`config/mask.png`）が含まれています。これを参考に作成してください。

### 3. Docker Composeでの起動

```bash
# ビルドと起動（バックグラウンド）
docker compose up --build -d

# ログの確認
docker compose logs -f

# 停止
docker compose down
```

### 4. FTP接続の設定

カメラからFTPで画像を送信する際の設定:

- **ホスト**: サーバーのIPアドレスまたはホスト名
- **ポート**: `21`
- **ユーザー名**: `ftpuser`
- **パスワード**: `ftpuser`
- **パッシブモード**: 有効（ポート21000-21010）

> **注意**: デフォルトの認証情報は本番環境では変更してください。`docker-compose.yml` のFTPサービスの `USERS` 環境変数を編集してください。

## システム構成

### Docker Composeサービス

- **backend**: FastAPIベースのバックエンドサービス（検知処理、API提供）
- **nginx**: フロントエンドの静的配信とAPIのリバースプロキシ
- **ftp**: FTPサーバー（カメラからの画像受信）

### ディレクトリ構造

```
SnowJamDetector/
├── config/          # 設定ファイル（settings.json, mask.png）
├── storage/         # 保存済み画像（自動生成）
│   └── archive/     # YYYY/MM/DD/ 形式で保存
├── ftp_data/        # FTP受信ディレクトリ（自動生成）
│   └── incoming/    # カメラからの画像受信先
└── logs/            # ログファイル（自動生成）
```

### ポート

- **80**: Web UI（HTTP）
- **21**: FTP（制御）
- **21000-21010**: FTP（パッシブモード）

> **注意**: ポート21を使用するため、既存のFTPサービスがある場合は停止してください。

## トラブルシューティング

### よくある問題

**Q: ポート80が既に使用されている**
- A: `docker-compose.yml` でポート番号を変更するか、既存のサービスを停止してください

**Q: FTP接続できない**
- A: ポート21が開いているか、ファイアウォールの設定を確認してください。ログは `docker compose logs ftp` で確認できます

**Q: 画像が検知されない**
- A: 設定のしきい値が適切か確認してください。低すぎると誤検知が増えます

**Q: Slack通知が送信されない**
- A: `slack_webhook_url` または `slack_bot_token` が正しく設定されているか確認してください

詳細なトラブルシューティングは [デプロイ手順書](docs/deployment.md#トラブルシューティング) を参照してください。

## 開発者向け情報

### ローカル開発（フロントエンドのみ）

```bash
cd frontend
npm install
npm run dev      # 開発サーバー起動
npm run build    # ビルド
npm run lint     # リント
```

### テスト

```bash
# バックエンドのテスト（実装予定）
# フロントエンドのテスト（実装予定）
```

### コーディング規約

- コードスタイル: ESLint/Prettier設定に従う
- コミットメッセージ: Conventional Commits形式
- 詳細は `docs/development-guide.md` を参照してください

## セキュリティに関する注意事項

⚠️ **重要**: 以下のファイルには機密情報が含まれる可能性があるため、Gitにコミットしないでください:

- `config/settings.json`（Slack認証情報など）
- `config/mask.png`（環境固有の設定）
- `.env` ファイル

これらのファイルは `.gitignore` に含まれています。テンプレートファイル（`.example` 付き）のみがリポジトリに含まれます。

## ドキュメント

- 📖 [デプロイ手順書](docs/deployment.md) - 詳細なデプロイ手順とトラブルシューティング
- 📋 [要件・設計書](docs/snowjam-detection-system.md) - システムの要件と設計
- 🔧 [技術スタック](docs/tech-stack.md) - 使用技術と実装ルール
- 🗺️ [ロードマップ](docs/roadmap.md) - 開発の進捗と計画
- 📝 [環境変数と設定](docs/environment.md) - 環境変数と設定ファイルの説明
- 👨‍💻 [開発ガイド](docs/development-guide.md) - 開発者向けガイドライン
- 📋 [変更履歴](CHANGELOG.md) - バージョンごとの変更履歴
- 🔒 [セキュリティポリシー](SECURITY.md) - セキュリティに関する情報

## ライセンス

[LICENSE](LICENSE) を参照してください。

## サポート

問題が発生した場合は、以下の情報を含めてIssueを作成してください:

- エラーメッセージ
- 実行環境（OS、Dockerバージョンなど）
- ログ（`docker compose logs` の出力）
