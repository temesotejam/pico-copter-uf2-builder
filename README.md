# Pico Copter UF2 Builder

アップロードされた`pico_copter-main.zip`のRaspberry Pi Pico用プログラムを、GitHub Actionsでコンパイルし、最新のUF2をGitHub Pagesからダウンロードするためのリポジトリです。

## 方針

- `source/`内の`.cpp`、`.hpp`、`.c`、`.h`は、アップロードされたZIPからそのままコピーしています。
- マイコンで動作する元プログラムは変更しません。
- ビルド前に`SOURCE_SHA256SUMS.txt`を検証し、元プログラムが変化している場合はWorkflowを停止します。
- 元のPico Copterリポジトリとは連携せず、このリポジトリだけで管理します。
- 元プロジェクトのMac固有Eigenパスは使用せず、Eigen 3.4.0をGitHub Actionsが別途取得します。

## 全体の流れ

```text
github.devまたはGitHubでコードを確認
→ GitHub ActionsでPico SDK・Eigenを準備
→ pico_copterをコンパイル
→ pico_copter.uf2を生成
→ GitHub Pagesへ最新UF2を公開
→ BOOTSELモードのPicoへコピー
```

## 初期設定

1. このリポジトリの`Settings`を開く
2. `Pages` → `Build and deployment`を開く
3. `Source`を`GitHub Actions`に設定する
4. `Actions`を開く
5. `Build Pico Copter UF2 and deploy Pages`を選ぶ
6. `Run workflow`を押す
7. 成功後、`Settings` → `Pages`に表示される公開先を開く

Workflowは`main`の対象ファイルを変更した場合にも自動実行されます。Pull Requestではコンパイル確認だけを行い、Pagesへの公開は行いません。

## 構成

```text
pico-copter-uf2-builder/
├─ source/                    元のPico Copterプロジェクト
├─ CMakeLists.txt             クラウドビルド用ラッパー
├─ SOURCE_SHA256SUMS.txt      元プログラムの整合性確認
├─ web/                       UF2配布ページ
└─ .github/workflows/
   └─ build-and-deploy.yml
```

`source/CMakeLists.txt`もZIPの内容を保存するため、そのまま残しています。GitHub Actionsではルートの`CMakeLists.txt`を使用し、次の元ビルド設定上の問題を外側から回避します。

- `CMAKE_C_STNDARD`という綴り誤り
- `/Users/itoukouhei/pico/eigen`という作者PC固有の絶対パス

## 依存関係

| 項目 | 固定値 |
|---|---|
| 対象ボード | `pico` |
| Pico SDK | `2.3.0` |
| Eigen | `3.4.0` |
| Eigenコミット | `3147391d946bb4b6c68edd901f2add6ac1f31f8c` |
| C++ | C++17 |

Eigenはサブモジュールにはせず、Workflow実行時に公式GitLabから指定コミットを取得し、GitHub Actionsのキャッシュへ保存します。

## 高速化

次の2種類をキャッシュします。

```text
Pico SDKキャッシュ
└─ SDK本体とサブモジュールを再利用

ccache
└─ C/C++のコンパイル済み中間結果を再利用
```

Eigenもバージョンとコミットをキーにしてキャッシュします。初回、キャッシュ削除後、依存バージョン変更後は通常どおり時間がかかります。

## ビルド結果

成功時は次を生成します。

```text
pico_copter.uf2
pico_copter.uf2.sha256
size.txt
manifest.json
configure.log
build.log
```

通常Artifactと診断ログの保持期間は1日です。GitHub Pagesには最後に成功したUF2が公開されます。

## 元コード内の問題について

元の`.cpp/.hpp/.c/.h`にコンパイルエラーやリンクエラーが見つかった場合、Workflowのログにそのまま表示されます。このリポジトリでは元コードを自動修正しません。必要な場合は、エラー箇所と修正案を確認してから別途判断します。

## UF2の書き込み

1. Pagesから`pico_copter-pico.uf2`をダウンロードする
2. Picoの`BOOTSEL`ボタンを押したままUSB接続する
3. 表示された`RPI-RP2`ドライブへUF2をコピーする
4. コピー後、Picoが自動的に再起動する
