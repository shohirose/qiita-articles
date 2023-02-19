<!--
title:   Window 10でのプログラミング環境構築
tags:    C++,MinGW,Python,msys2
id:      299c5fbd056c4e96b6bd
private: true
-->
# MSYS2

## 環境構築手順

[MSYS2](https://www.msys2.org/)から64bit版インストーラ（`msys2-x86_64-[yyyymmdd].exe`）を入手する。

インストール後、MSYS2を起動し、

```terminal
$ pacman -Syu
```

をタイプしてパッケージを更新する。

## パッケージのインストール

各種パッケージをインストールするには、`pacman`を使えばいい。`pacboy`を使うと`mingw32`と`mingw64`を区別してインストールするのが楽になる。例えば`toolchain`パッケージをインストールするときに

```terminal:pacboy-example
$ pacboy -S toolchain:m
```

とすれば、`mingw32`と`mingw64`の両方の`toolchain`がインストールされる。

- `m`: `mingw32`と`mingw64`の両方
- `x`: `mingw64`
- `i`: `mingw32`

とりあえずインストールするパッケージは以下の通り。

- `toolchain`
- `git`
- `cmake`
- `eigen3`
  - `boost`
  - `fftw`
  - `freeglut`
  - `suitesparse`
- `gtest`
- `openblas`
- `lapack`
- `arpack`