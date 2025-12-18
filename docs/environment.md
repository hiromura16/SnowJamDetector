# 環境変数と設定ファイル

このドキュメントでは、スノージャム検知システムで使用する環境変数と設定ファイルについて説明します。

## 設定ファイル

### config/settings.json

メインの設定ファイルです。テンプレートは `config/settings.json.example` を参照してください。

#### 設定項目

| 項目 | 型 | デフォルト | 説明 |
|------|-----|-----------|------|
| `threshold` | float | 0.15 | 検知のしきい値（0.0-1.0）。低いほど敏感に反応します |
| `consecutive_hits` | int | 3 | 連続検知回数。この回数連続で検知された場合にアラートを発報 |
| `binary_threshold` | int | 30 | 二値化の閾値（0-255） |
| `blur_kernel` | int | 3 | ぼかしカーネルサイズ（ノイズ除去用） |
| `overlay_color` | string | "#ff69b4" | オーバーレイの色（HEX形式） |
| `overlay_alpha` | float | 0.35 | オーバーレイの不透明度（0.0-1.0） |
| `delay_monitor_enabled` | boolean | true | 遅延監視の有効/無効 |
| `delay_threshold_seconds` | int | 300 | 遅延警告の閾値（秒）。この時間を超えて画像が到着しない場合に警告 |
| `alarm_enabled` | boolean | true | 警報の有効/無効 |
| `slack_webhook_url` | string | "" | Slack通知のWebhook URL（オプション） |
| `slack_bot_token` | string | "" | Slack Bot Token（画像添付に必要、オプション） |
| `slack_channel` | string | "" | Slack通知先チャンネル（オプション） |
| `mask_inclusive` | boolean | true | マスクの適用モード。true=白領域のみ検出、false=黒領域のみ検出 |
| `gpio_pin` | int/null | 17 | GPIOピン番号（Raspberry Piなどで使用、オプション） |

#### 設定例

```json
{
  "threshold": 0.15,
  "consecutive_hits": 3,
  "binary_threshold": 30,
  "blur_kernel": 3,
  "overlay_color": "#ff69b4",
  "overlay_alpha": 0.35,
  "delay_monitor_enabled": true,
  "delay_threshold_seconds": 300,
  "alarm_enabled": true,
  "slack_webhook_url": "https://hooks.slack.com/services/YOUR/WEBHOOK/URL",
  "slack_bot_token": "xoxb-your-bot-token",
  "slack_channel": "#snowjam-alerts",
  "mask_inclusive": true,
  "gpio_pin": 17
}
```

### config/mask.png

検知範囲を限定するためのマスク画像です。

- **形式**: PNGまたはJPEG（モノクロ推奨）
- **白い領域**: 検知対象（`mask_inclusive: true` の場合）
- **黒い領域**: 検知対象外（`mask_inclusive: true` の場合）

マスクファイルがない場合は、画像全体が検知対象になります。

## Docker Compose環境変数

### Backendサービス

`docker-compose.yml` の `backend` サービスで使用される環境変数:

| 環境変数 | デフォルト | 説明 |
|---------|-----------|------|
| `SETTINGS_PATH` | `/config/settings.json` | 設定ファイルのパス |
| `MASK_PATH` | `/config/mask.png` | マスクファイルのパス |
| `STORAGE_ROOT` | `/storage/archive` | 画像保存先のルートディレクトリ |
| `FTP_INCOMING` | `/ftp_data/incoming` | FTP受信ディレクトリ |
| `LOG_ROOT` | `/logs` | ログファイルの保存先 |

### FTPサービス

| 環境変数 | デフォルト | 説明 |
|---------|-----------|------|
| `USERS` | `ftpuser\|ftpuser\|/ftp_data/incoming` | FTPユーザー設定（`ユーザー名\|パスワード\|ホームディレクトリ`） |
| `ADDRESS` | `0.0.0.0` | バインドアドレス |

#### FTPユーザーの変更例

`docker-compose.yml` を編集:

```yaml
ftp:
  environment:
    - USERS=myuser|mypassword|/ftp_data/incoming
```

複数のユーザーを設定する場合:

```yaml
ftp:
  environment:
    - USERS=user1|pass1|/ftp_data/incoming user2|pass2|/ftp_data/incoming
```

## Slack通知の設定

### Webhook URLのみの場合

画像添付なしのテキスト通知のみが送信されます:

```json
{
  "slack_webhook_url": "https://hooks.slack.com/services/YOUR/WEBHOOK/URL",
  "slack_bot_token": "",
  "slack_channel": ""
}
```

### Bot Tokenを設定した場合

画像添付付きの通知が送信されます:

```json
{
  "slack_webhook_url": "",
  "slack_bot_token": "xoxb-your-bot-token",
  "slack_channel": "#snowjam-alerts"
}
```

### Webhook URLとBot Tokenの両方を設定した場合

Bot Tokenが優先され、画像添付付きの通知が送信されます。

### Slack Appの設定方法

1. [Slack API](https://api.slack.com/apps) でアプリを作成
2. **Incoming Webhooks** を有効化してWebhook URLを取得
3. **OAuth & Permissions** でBot Tokenを取得
4. Bot Tokenには以下のスコープが必要:
   - `files:write`（画像アップロード用）
   - `chat:write`（メッセージ送信用）

## GPIO設定（Raspberry Piなど）

GPIOを使用する場合は、以下の設定が必要です:

1. `config/settings.json` で `gpio_pin` を設定
2. DockerコンテナにGPIOデバイスへのアクセス権限を付与

`docker-compose.yml` に以下を追加:

```yaml
backend:
  devices:
    - /dev/gpiomem:/dev/gpiomem
  # または
  privileged: true  # セキュリティ上の注意が必要
```

## 設定の変更方法

### 方法1: 設定ファイルを直接編集

1. `config/settings.json` を編集
2. バックエンドサービスを再起動:

```bash
docker compose restart backend
```

### 方法2: Web UIから変更

1. ブラウザで `http://localhost` にアクセス
2. 設定画面（`/settings`）で設定を変更
3. 変更は自動的に `config/settings.json` に保存されます

## 設定のバックアップ

重要な設定をバックアップする場合:

```bash
# 設定ファイルのバックアップ
tar -czf backup-config-$(date +%Y%m%d).tar.gz config/
```

## セキュリティに関する注意事項

⚠️ **重要**: 以下の情報は機密情報です。Gitにコミットしないでください:

- `slack_webhook_url`
- `slack_bot_token`
- FTPの認証情報（`docker-compose.yml` の `USERS`）

これらの情報は `.gitignore` に含まれていますが、設定ファイルを共有する際は注意してください。

