# クイックスタートガイド

## 初回セットアップ（コピペ可能）

```bash
# 1. aquaでツールをインストール
./.aqua/bin/aqua i

# 2. PATHに追加（オプション、以降のコマンドで ./aqua/bin/ を省略できる）
export PATH="$PWD/.aqua/bin:$PATH"

# 3. 既存Webアプリを ./web に配置
# （例：git clone <repo-url> web）

# 4. Webアプリの依存関係をインストール
pnpm -C web install

# 5. 開発サーバーのポートとビルド出力ディレクトリを確認
# web/package.json を確認して、以下を特定：
#   - 開発サーバーのポート（例: 5173, 3000）
#   - ビルド出力ディレクトリ（例: dist, build, out）

# 6. src-tauri/tauri.conf.json を編集（必要に応じて）
#   - build.devPath: 開発サーバーのポートに合わせる
#   - build.distDir: ビルド出力ディレクトリに合わせる
```

## 開発モード（ホットリロード対応）

**推奨方法（1コマンドで起動）**:

```bash
cd src-tauri
cargo tauri dev
```

このコマンドで：
- Webアプリの開発サーバーが自動起動
- Tauriアプリが起動
- ホットリロードが有効（ファイル変更時に自動リロード）

**手動で起動する場合**:

```bash
# ターミナル1: Webアプリの開発サーバーを起動
cd web/lottie-tools/web-editor
pnpm dev

# ターミナル2: Tauriアプリを起動
cd src-tauri
cargo tauri dev
```

## 本番ビルド

```bash
# 1. Webアプリをビルド
pnpm -C web build

# 2. Tauriアプリをビルド
./.aqua/bin/tauri build
# または PATH追加済みの場合: tauri build

# ビルド成果物は以下に出力されます
# src-tauri/target/release/bundle/
```

## よくある設定変更

### ポート番号の変更

`src-tauri/tauri.conf.json` を編集：

```json
{
  "build": {
    "devPath": "http://localhost:3000"  // 実際のポートに変更
  }
}
```

### ビルド出力ディレクトリの変更

`src-tauri/tauri.conf.json` を編集：

```json
{
  "build": {
    "distDir": "../web/build"  // 実際の出力ディレクトリに変更
  }
}
```
