# デプロイ手順書

このドキュメントでは、スノージャム検知システムを初めてデプロイする際の手順を詳しく説明します。

## 前提条件

- Docker と Docker Compose がインストールされていること
- システム要件:
  - Docker 20.10以上
  - Docker Compose 2.0以上
  - ディスク容量: 最低10GB（画像保存用）
  - メモリ: 最低2GB

## 初回セットアップ手順

### 1. リポジトリのクローン

```bash
git clone <repository-url>
cd SnowJamDetector
```

### 2. 設定ファイルの準備

#### 2.1 設定ファイルのコピー

```bash
# 設定ファイルのテンプレートをコピー
cp config/settings.json.example config/settings.json
```

#### 2.2 設定ファイルの編集

`config/settings.json` を編集して、以下の項目を設定します:

- **threshold**: 検知のしきい値（0.0-1.0、デフォルト: 0.15）
- **consecutive_hits**: 連続検知回数（デフォルト: 3）
- **binary_threshold**: 二値化の閾値（0-255、デフォルト: 30）
- **blur_kernel**: ぼかしカーネルサイズ（デフォルト: 3）
- **delay_monitor_enabled**: 遅延監視の有効/無効（デフォルト: true）
- **delay_threshold_seconds**: 遅延警告の閾値（秒、デフォルト: 300）
- **alarm_enabled**: 警報の有効/無効（デフォルト: true）
- **slack_webhook_url**: Slack通知のWebhook URL（オプション）
- **slack_bot_token**: Slack Bot Token（画像添付に必要、オプション）
- **slack_channel**: Slack通知先チャンネル（オプション）
- **gpio_pin**: GPIOピン番号（Raspberry Piなどで使用、オプション）

#### 2.3 マスクファイルの準備（オプション）

検知範囲を限定したい場合は、モノクロのPNGまたはJPEG画像を `config/mask.png` として配置してください。
- **白い領域**: 検知対象（`mask_inclusive: true` の場合）
- **黒い領域**: 検知対象外（`mask_inclusive: true` の場合）

マスクファイルがない場合は、画像全体が検知対象になります。

**マスク画像の作成方法**:
1. カメラから取得した画像と同じサイズの画像を用意
2. 画像編集ソフト（GIMP、Photoshop、ペイントなど）で編集
3. 検知したい領域を**白**、検知したくない領域を**黒**で塗りつぶす
4. モノクロ（グレースケール）で保存（PNGまたはJPEG形式）
5. `config/mask.png` として保存

リポジトリにはサンプルマスク画像（`config/mask.png`）が含まれています。これを参考に作成してください。

### 3. ディレクトリの作成

以下のディレクトリが自動的に作成されますが、事前に作成することもできます:

```bash
mkdir -p ftp_data/incoming
mkdir -p storage/archive
mkdir -p logs
```

### 4. FTPポートの確認

FTPサービスはポート21を使用します。既存のFTPサービスが動作している場合は、停止してください:

```bash
# ポート21が使用されているか確認
sudo lsof -i :21

# 既存のFTPサービスを停止（例: vsftpd）
sudo systemctl stop vsftpd
```

### 5. Docker Composeでの起動

```bash
# ビルドと起動
docker compose up --build -d

# ログの確認
docker compose logs -f
```

### 6. 動作確認

#### 6.1 ヘルスチェック

```bash
curl http://localhost/health
```

正常な場合は以下のようなJSONが返されます:

```json
{
  "status": "ok",
  "timestamp": "2024-01-01T00:00:00Z",
  "storage": {
    "total": 1000000000,
    "used": 100000000,
    "free": 900000000,
    "path": "/storage/archive"
  },
  "delay_monitor": {
    "enabled": true,
    "threshold_seconds": 300,
    "seconds_since_last_image": null
  }
}
```

#### 6.2 Web UIの確認

ブラウザで `http://localhost` にアクセスして、ダッシュボードが表示されることを確認してください。

#### 6.3 FTP接続の確認

FTPクライアントで以下の情報で接続を試みます:

- ホスト: `localhost`（またはサーバーのIPアドレス）
- ポート: `21`
- ユーザー名: `ftpuser`
- パスワード: `ftpuser`

接続できたら、画像ファイルをアップロードして、システムが検知することを確認してください。

## 運用中の操作

### サービスの停止

```bash
docker compose down
```

### サービスの再起動

```bash
docker compose restart
```

### ログの確認

```bash
# すべてのサービスのログ
docker compose logs -f

# 特定のサービスのログ
docker compose logs -f backend
docker compose logs -f nginx
docker compose logs -f ftp
```

### 設定の変更

1. `config/settings.json` を編集
2. バックエンドサービスを再起動:

```bash
docker compose restart backend
```

または、Web UIの設定画面から変更することもできます。

## トラブルシューティング

### ポートが既に使用されている

エラー: `Bind for 0.0.0.0:80 failed: port is already allocated`

**解決方法**: ポート80を使用しているサービスを停止するか、`docker-compose.yml` でポート番号を変更してください。

### FTP接続できない

**確認事項**:
1. ポート21が開いているか確認
2. ファイアウォールの設定を確認
3. FTPサービスのログを確認: `docker compose logs ftp`

### 画像が検知されない

**確認事項**:
1. FTPで画像が正常にアップロードされているか確認
2. `ftp_data/incoming` ディレクトリに画像が存在するか確認
3. `storage/archive` ディレクトリに画像が保存されているか確認
4. 設定のしきい値が適切か確認（低すぎると誤検知が増えます）

### ストレージが不足している

90日を超えた画像は自動的に削除されますが、手動で削除する場合は:

```bash
# 古い画像を確認
find storage/archive -type f -mtime +90

# 古い画像を削除（注意: 実行前に確認してください）
find storage/archive -type f -mtime +90 -delete
```

### Slack通知が送信されない

**確認事項**:
1. `slack_webhook_url` または `slack_bot_token` が正しく設定されているか
2. SlackのWebhook URLまたはBot Tokenが有効か
3. バックエンドのログを確認: `docker compose logs backend`

### GPIOが動作しない

**確認事項**:
1. Raspberry PiなどのGPIO対応デバイスで実行しているか
2. GPIOピン番号が正しく設定されているか
3. 必要な権限があるか（通常はroot権限が必要）

## バックアップとリストア

### バックアップ

重要なデータをバックアップする場合:

```bash
# 設定ファイルのバックアップ
tar -czf backup-config-$(date +%Y%m%d).tar.gz config/

# 保存済み画像のバックアップ（オプション）
tar -czf backup-storage-$(date +%Y%m%d).tar.gz storage/
```

### リストア

```bash
# 設定ファイルのリストア
tar -xzf backup-config-YYYYMMDD.tar.gz
```

## セキュリティに関する注意事項

1. **FTP認証情報**: デフォルトの `ftpuser/ftpuser` は本番環境では変更してください
2. **Slack認証情報**: Webhook URLやBot Tokenは機密情報です。Gitにコミットしないでください
3. **ネットワーク**: 本番環境では、ファイアウォールで適切にポートを制限してください
4. **設定ファイル**: `config/settings.json` には機密情報が含まれる可能性があるため、Gitにコミットしないでください

## 次のステップ

- システムの詳細な設定方法は `docs/snowjam-detection-system.md` を参照してください
- 技術的な詳細は `docs/tech-stack.md` を参照してください
- 開発ガイドは `AGENTS.md` を参照してください

