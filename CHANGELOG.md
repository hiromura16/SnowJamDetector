# Changelog

このプロジェクトのすべての重要な変更は、このファイルに記録されます。

形式は [Keep a Changelog](https://keepachangelog.com/ja/1.0.0/) に基づいており、
このプロジェクトは [Semantic Versioning](https://semver.org/lang/ja/) に準拠しています。

## [0.3.0] - 2024-12-18

### 追加
- GitHub管理用の設定とドキュメントを整備
- デプロイ手順書（`docs/deployment.md`）を追加
- 環境変数と設定ファイルの説明書（`docs/environment.md`）を追加
- バージョン管理ファイル（`VERSION`）を追加
- `.editorconfig` を追加
- Docker Composeにヘルスチェックとネットワーク設定を追加

### 変更
- README.mdを初心者向けに改善
- Markdownファイル名を統一（小文字 + ハイフン区切り）
- `.gitignore` を改善し、機密情報を含む設定ファイルを除外

### セキュリティ
- 設定ファイルのテンプレート（`config/settings.json.example`）を追加
- 機密情報を含むファイルをGit管理から除外

## [0.2.0] - 2024-12-02

### 追加
- フロントエンド設定画面の実装
- マスク画像のアップロード機能
- 設定のインポート/エクスポート機能

## [0.1.0] - 2024-11-01

### 追加
- 初回リリース
- 画像差分によるスノージャム検知機能
- Slack通知機能
- GPIO出力機能
- Web UIダッシュボード

[0.3.0]: https://github.com/yourusername/SnowJamDetector/compare/v0.2.0...v0.3.0
[0.2.0]: https://github.com/yourusername/SnowJamDetector/compare/v0.1.0...v0.2.0
[0.1.0]: https://github.com/yourusername/SnowJamDetector/releases/tag/v0.1.0

