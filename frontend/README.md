# スノージャム検知システム フロントエンド

React + TypeScript + Vite で構築されたWeb UIです。

## 開発

### セットアップ

```bash
npm install
```

### 開発サーバー起動

```bash
npm run dev
```

開発サーバーは `http://localhost:5173` で起動します（ポートはViteの設定に依存）。

### ビルド

```bash
npm run build
```

ビルド結果は `dist/` ディレクトリに出力されます。

### リント

```bash
npm run lint
```

### プレビュー

ビルド済みのファイルをプレビュー:

```bash
npm run preview
```

## プロジェクト構造

```
frontend/
├── src/
│   ├── App.tsx          # メインアプリケーションコンポーネント
│   ├── App.css          # アプリケーションスタイル
│   ├── main.tsx         # エントリーポイント
│   └── index.css        # グローバルスタイル
├── public/              # 静的アセット
└── package.json         # 依存関係とスクリプト
```

## Docker Composeでのビルド

フロントエンドは `nginx/Dockerfile` で自動的にビルドされ、Nginxコンテナに含まれます。

詳細はプロジェクトルートの [README.md](../README.md) を参照してください。
