<!--
title:   Ninjaの速さを体感するともはやMakeには戻れない
tags:    C++,Make,Ninja
id:      a1e2fcb1ea25fb4e7644
private: false
-->
# はじめに

[Ninja](https://ninja-build.org/)というビルドシステムがあるのは知っていたのですが今まで使っていませんでした。
先日、研究で使用しているライブラリを最新版にアップデートする時、試しにMakeの代わりにNinjaを使ってみたら爆速で、現在はデフォルトでNinjaを使用するようになりました。[VTKライブラリ](https://www.vtk.org/)をビルドするとき、__Makeだと少なくとも10分以上掛かっていたのがNinjaでは1分強だった__ので、これだけスピードが違うともうMakeを使う理由がないですね。特に大きなライブラリをビルドするとき顕著に差が出ると感じています。

# 使い方

[バイナリ](https://ninja-build.org/)をダウンロードしローカルディレクトリに置いてパスを通すか、[パッケージマネージャを通してインストール](https://github.com/ninja-build/ninja/wiki/Pre-built-Ninja-packages)します。

CMakeを使っていれば、簡単にMakeとNinjaを切り替えられます。
以下のように`-GNinja`を追加してconfigureしてください。

```terminal:Makeの場合
mkdir build
cd build
cmake ..
make && make install
```

```terminal:Ninjaの場合
mkdir build
cd build
cmake -GNinja ..
ninja && ninja install
```