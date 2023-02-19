<!--
title:   [OpenFOAM/foam-extend] wmakeのコンパイルフラッグを変更する
tags:    OpenFOAM,foam-extend,wmake
id:      c82e4cde45623bf973d5
private: false
-->
# はじめに

OpenFOAM/foam-extendはwmakeという独自のビルドシステム（中身はただのbashファイル）を使っています。
wmakeには`Opt`/`Debug`/`Prof`という3種類のビルドが用意されているのですが、コンパイルフラッグを追加したり、各モードに既定のグローバルなコンパイルフラッグを変更したいときがあります。

今回は、研究室の同僚のプログラムをcallgrindを使ってプロファイリングするために、`Prof`に既定されたグローバルなコンパイルフラッグ`-pg -O2`を変更する方法を見つけるのに苦労したので、忘れないようにメモとして残しておきます。

wmakeはOpenFOAM/foam-extendでしか使われておらず、ドキュメントも充実していないので、同じように苦労された方々が多いんじゃないでしょうか。

# グローバルなコンパイルオプションの設定場所


`OpenFOAM-#.#`/`foam-extend-#.#`のディレクトリ直下に`wmake`というディレクトリがあります。その中の`rules`というディレクトリの下に、各プラットフォームおよびコンパイラごとの設定ファイルが置かれています。
今回はUbuntu-16.04LTS、gcc、Profモードのオプションを変更したかったので、`wmake/rules/linux64Gcc/c++Prof`を変更しました。

```bash:c++Prof：変更前
c++DBUG    = -pg
c++OPT     = -O2
```

```bash:c++Prof：変更後
c++DBUG    = -g
c++OPT     = -O2
```

# ライブラリ・アプリケーションごとのコンパイルオプションの設定

各ライブラリ・アプリケーションフォルダ直下の`Make`ディレクトリにあるoption/option.debugファイルの`EXE_INC`にコンパイルフラッグを追加することができます。`Opt`/`Prof`モードでは`option`ファイルを、`Debug`モードでは`option.debug`ファイルが使われます。
特定のライブラリ・アプリケーションのみ設定を一時的に変更したい場合に便利です。

```bash:Make/option：変更前
EXE_INC = \
  -I$(LIB_SRC)/meshTools/lnInclude \
  ...

LIB_LIBS = \
  -lmeshTools \
  ...
```

```bash:Make/option：変更後
EXE_INC = \
  -Wall -Wpedantic \
  -I$(LIB_SRC)/meshTools/lnInclude \
  ...

LIB_LIBS = \
  -lmeshTools \
  ...
```