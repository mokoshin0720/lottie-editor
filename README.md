# Lottie Editor - macOS App

Lottie Open StudioをTauri v2でmacOSアプリ化したプロジェクトです。

## 前提条件

- macOS
- [aqua](https://aquaproj.github.io/) がインストールされていること
  ```bash
  brew install aquaproj/aqua/aqua
  ```

## セットアップ

### 1. ツールのインストール（aqua）

```bash
# aquaでツールをインストール（Node.js、pnpm、tauri-cli）
./.aqua/bin/aqua i
```

または、PATHに追加して使用：

```bash
export PATH="$PWD/.aqua/bin:$PATH"
aqua i
```

### 2. 既存Webアプリの配置

既存のLottie Open StudioのWebアプリを `./web` ディレクトリに配置してください。

```bash
# 例：GitHubからクローンする場合
git clone <lottie-open-studio-repo-url> web
cd web
# 既存の依存関係をインストール（必要に応じて）
```

### 3. Webアプリの依存関係インストール

```bash
pnpm -C web install
```

### 4. 開発サーバーのポート確認

Webアプリの `package.json` を確認し、開発サーバーのポートを特定してください。
一般的には以下のいずれかです：
- Vite: `5173` (デフォルト)
- Create React App: `3000`
- Next.js: `3000`

ポートが異なる場合は、`src-tauri/tauri.conf.json` の `build.devPath` を修正してください。

### 5. ビルド出力ディレクトリの確認

Webアプリの `package.json` の `build` スクリプトを確認し、出力ディレクトリを特定してください。
一般的には以下のいずれかです：
- `dist`
- `build`
- `out`
- `.next/out` (Next.jsの場合)

出力ディレクトリが異なる場合は、`src-tauri/tauri.conf.json` の `build.distDir` を修正してください。

## 開発

### 開発モード（開発サーバーを使用）

```bash
# 1. Webアプリの開発サーバーを起動（別ターミナル）
pnpm -C web dev

# 2. Tauriアプリを起動（別ターミナル）
./.aqua/bin/tauri dev
```

または、PATHに追加済みの場合：

```bash
pnpm -C web dev
tauri dev
```

### 本番ビルド

```bash
# 1. Webアプリをビルド
pnpm -C web build

# 2. Tauriアプリをビルド
./.aqua/bin/tauri build
```

ビルド成果物は `src-tauri/target/release/bundle/` に出力されます。

## ディレクトリ構成

```
lottie-editor/
├── .aqua/              # aquaでインストールされたツール
├── web/                # 既存のWebアプリ（ここに配置）
│   ├── package.json
│   ├── dist/          # ビルド出力（例）
│   └── ...
├── src-tauri/          # Tauriプロジェクト
│   ├── Cargo.toml
│   ├── tauri.conf.json
│   ├── src/
│   │   └── main.rs
│   └── ...
├── aqua.yaml           # aqua設定
├── .gitignore
└── README.md
```

## 設定ファイル

### aqua.yaml

Node.js、pnpm、tauri-cliのバージョンを管理します。

### src-tauri/tauri.conf.json

Tauriアプリの設定ファイルです。主な設定項目：

- `build.devPath`: 開発時のURL（例: `http://localhost:5173`）
- `build.distDir`: 本番ビルド時の出力ディレクトリ（例: `../web/dist`）
- `build.beforeDevCommand`: 開発サーバー起動コマンド
- `build.beforeBuildCommand`: ビルド前のコマンド

## トラブルシューティング

### ポート不一致エラー

**症状**: Tauriアプリが起動しない、または空白画面が表示される

**原因**: `tauri.conf.json` の `devPath` と実際の開発サーバーのポートが一致していない

**対処**:
1. Webアプリの開発サーバーのポートを確認
2. `src-tauri/tauri.conf.json` の `build.devPath` を修正

```json
{
  "build": {
    "devPath": "http://localhost:5173"  // 実際のポートに合わせる
  }
}
```

### distDir不一致エラー

**症状**: `tauri build` でエラー、またはビルド後のアプリが空白画面

**原因**: `tauri.conf.json` の `distDir` と実際のビルド出力ディレクトリが一致していない

**対処**:
1. `pnpm -C web build` を実行して出力ディレクトリを確認
2. `src-tauri/tauri.conf.json` の `build.distDir` を修正

```json
{
  "build": {
    "distDir": "../web/dist"  // 実際の出力ディレクトリに合わせる
  }
}
```

### CSP（Content Security Policy）エラー

**症状**: 一部のリソースが読み込まれない、コンソールにCSPエラー

**原因**: TauriのデフォルトCSPが厳しすぎる

**対処**: `src-tauri/tauri.conf.json` でCSPを緩和（既に `null` に設定済み）

```json
{
  "app": {
    "security": {
      "csp": null  // 開発時は無効化（本番では適切に設定）
    }
  }
}
```

### WASM/Worker関連エラー

**症状**: WASMやWeb Workerが動作しない

**原因**: WebViewの制約やパスの問題

**対処**:
1. 開発モードでは通常問題なし（`http://localhost` 経由）
2. 本番ビルドで問題が出る場合：
   - `tauri.conf.json` の `build.distDir` が正しいか確認
   - Webアプリのビルド設定でbase pathを確認
   - 必要に応じて `app.security.csp` を調整

### ビルド出力が空

**症状**: `pnpm -C web build` を実行しても出力ディレクトリが空

**原因**: ビルドスクリプトが正しく実行されていない、または出力先が異なる

**対処**:
1. `web/package.json` の `build` スクリプトを確認
2. 手動で `pnpm -C web build` を実行して出力を確認
3. 出力ディレクトリを `tauri.conf.json` の `distDir` に反映

### Rust/Cargo関連エラー

**症状**: `tauri dev` や `tauri build` でRustのコンパイルエラー

**原因**: Rustツールチェーンが未インストール、またはバージョン不一致

**対処**:
```bash
# Rustをインストール（未インストールの場合）
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# ツールチェーンを更新
rustup update
```

### 開発サーバーが起動しない

**症状**: `pnpm -C web dev` でエラー

**原因**: 依存関係が未インストール、またはポートが既に使用中

**対処**:
```bash
# 依存関係を再インストール
pnpm -C web install

# ポートが使用中の場合、別のポートを指定
# Viteの場合: pnpm -C web dev --port 5174
```

## アイコンについて

本番ビルド（`tauri build`）にはアイコンファイルが必要です。

アイコンファイルを `src-tauri/icons/` に配置してください：
- `32x32.png`
- `128x128.png`
- `128x128@2x.png`
- `icon.icns` (macOS用)

アイコンがない場合、`src-tauri/tauri.conf.json` の `bundle.icon` を空配列に変更してください：

```json
{
  "bundle": {
    "icon": []
  }
}
```

開発モード（`tauri dev`）ではアイコンは不要です。

## 注意事項

- このプロジェクトは「現行移植のみ」を目的としています
- 機能追加・UI改変は行いません
- ネイティブ機能（ファイル保存、ドラッグ＆ドロップ等）は実装していません
- 配布/署名/notarizationは不要です（ローカル開発のみ）
