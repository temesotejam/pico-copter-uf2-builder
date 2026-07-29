# Pico Copter firmware

このフォルダが、Raspberry Pi Picoへ書き込むファームウェアの独立したプロジェクトです。共有するときは`firmware/pico_copter`フォルダをそのまま取り出せます。

`.cpp`、`.hpp`、`.c`、`.h`は`github.dev`または通常のローカル開発環境で編集できます。変更履歴はGitで管理され、GitHub Actionsが変更後のファームウェアをコンパイルします。

## 必要なもの

- Raspberry Pi Pico SDK 2.3.0
- ARM GCC
- CMake 3.13以上
- Eigen 3.4.0

Eigenの場所を指定しない場合、CMakeが公式Eigenの固定コミットを自動取得します。

## ビルド例

```bash
cmake -S . -B build \
  -DPICO_SDK_PATH=/path/to/pico-sdk \
  -DPICO_BOARD=pico \
  -DCMAKE_BUILD_TYPE=Release

cmake --build build --parallel
```

生成物は`build/pico_copter.uf2`です。

既にEigenを持っている場合は、次を追加できます。

```bash
-DEIGEN3_INCLUDE_DIR=/path/to/eigen
```
