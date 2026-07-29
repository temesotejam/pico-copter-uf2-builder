# Pico Copter UF2 Builder

Raspberry Pi Pico用のPico Copterファームウェアを、GitHub Actionsでコンパイルし、最新のUF2をGitHub Pagesからダウンロードするためのリポジトリです。

## ファームウェアとビルダーの分離

マイコンへ書き込むプログラムは、次の1フォルダへまとめています。

```text
firmware/pico_copter/
```

他人と共有するときは、このフォルダをそのまま取り出せます。フォルダ内には、プログラム本体、独立ビルド用`CMakeLists.txt`、ビルド手順、ソース整合性ファイルが入っています。

```text
firmware/pico_copter/
├─ pico_copter.cpp / .hpp
├─ control.cpp / .hpp
├─ ekf.cpp / .hpp
├─ sensor.cpp / .hpp
├─ pwm.cpp / .hpp
├─ radio.cpp / .hpp
├─ lsm9ds1_reg.c / .h
├─ CMakeLists.txt
├─ SOURCE_SHA256SUMS.txt
└─ README.md
```

`.cpp`、`.hpp`、`.c`、`.h`は、提供された元プログラムから内容を変更していません。GitHub Actionsは毎回`SOURCE_SHA256SUMS.txt`を検証し、プログラムファイルが意図せず変化している場合はビルドを停止します。

GitHub Actions、キャッシュ、Web配布画面などは、ファームウェアフォルダの外側に分離しています。

## リポジトリ構成

```text
pico-copter-uf2-builder/
├─ firmware/
│  └─ pico_copter/              共有・持ち出し可能なファームウェア
├─ web/                         UF2配布ページ
├─ .github/workflows/
│  └─ build-and-deploy.yml      クラウドビルドとPages公開
├─ .gitignore
└─ README.md
```

元のPico Copterリポジトリとのサブモジュール連携や自動同期はありません。このリポジトリ内のファイルだけで管理します。

## GitHubでUF2を生成する

1. `Settings`を開く
2. `Pages` → `Build and deployment`を開く
3. `Source`を`GitHub Actions`に設定する
4. `Actions`を開く
5. `Build Pico Copter UF2 and deploy Pages`を選ぶ
6. `Run workflow`を押す
7. 成功後、`Settings` → `Pages`に表示される公開先を開く

`main`のファームウェアやWorkflowを変更した場合も自動実行されます。Pull Requestではコンパイル確認だけを行い、Pagesへの公開は行いません。

## ファームウェアフォルダ単体でビルドする

`firmware/pico_copter`へ移動し、Pico SDKを指定してビルドします。

```bash
cd firmware/pico_copter

cmake -S . -B build \
  -DPICO_SDK_PATH=/path/to/pico-sdk \
  -DPICO_BOARD=pico \
  -DCMAKE_BUILD_TYPE=Release

cmake --build build --parallel
```

生成物は次です。

```text
firmware/pico_copter/build/pico_copter.uf2
```

## 依存関係

| 項目 | 固定値 |
|---|---|
| 対象ボード | `pico` |
| Pico SDK | `2.3.0` |
| Eigen | `3.4.0` |
| Eigenコミット | `3147391d946bb4b6c68edd901f2add6ac1f31f8c` |
| C++ | C++17 |

ファームウェアフォルダのCMakeは、`EIGEN3_INCLUDE_DIR`が指定されていない場合、公式Eigenの固定コミットを自動取得します。既にEigenを持っている場合は、次のように指定できます。

```bash
-DEIGEN3_INCLUDE_DIR=/path/to/eigen
```

GitHub ActionsではEigenを別に取得してキャッシュし、そのパスをファームウェア側のCMakeへ渡します。

## 高速化

GitHub Actionsでは次をキャッシュします。

```text
Pico SDK
Eigen 3.4.0
ccacheによるC/C++のコンパイル結果
```

初回、キャッシュ削除後、依存バージョン変更後は通常どおり時間がかかります。

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

元の`.cpp/.hpp/.c/.h`にコンパイルエラーやリンクエラーが見つかった場合、Workflowのログへそのまま表示します。プログラム本体を自動修正することはありません。

## UF2の書き込み

1. Pagesから`pico_copter-pico.uf2`をダウンロードする
2. Picoの`BOOTSEL`ボタンを押したままUSB接続する
3. 表示された`RPI-RP2`ドライブへUF2をコピーする
4. コピー後、Picoが自動的に再起動する
