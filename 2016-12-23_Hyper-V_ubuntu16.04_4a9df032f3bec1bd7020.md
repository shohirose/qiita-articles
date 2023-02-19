<!--
title:   Windows10Pro Hyper-VにUbuntu 16.04 LTSをインストール
tags:    Hyper-V,ubuntu16.04
id:      4a9df032f3bec1bd7020
private: false
-->
# 参考ウェブページ

[Windows10でクライアントHyper-Vを使ってみる。セキュアブートの無効化は忘れずに！](http://i-think-it.net/windows10-hyper-v-1/)
[Hyper-V Virtual Machine Connection](https://technet.microsoft.com/en-us/windows-server-docs/compute/hyper-v/learn-more/hyper-v-virtual-machine-connect)
[モバイル環境では外部アダプタを使わない方がいい](http://nullpo-head.hateblo.jp/entry/2013/11/26/025119)

## 注意すること

- インストールした仮想マシンの設定において、セキュアブートの無効化をしないとOSを起動できない。
- VMwarePlayerとHyper-Vは共存できない。
- ~~有線LAN、無線LANの各ハードウェアごとに外部接続用の仮想化スイッチを作成する必要がある。~~
- 外部仮想スイッチを作成するとネットワークアダプタに関するエラーが起こりネットワークに接続できなくなる。解決方法は以下の通り。
 1. 外部ではなく内部仮想スイッチを作成する。
 2. コントロールパネル-->ネットワークとインターネット-->ネットワーク接続から使用しているWifiや有線LAN回線のプロパティを選択する。
 3. インターネット接続の共有にチェックをいれる。
- 仮想マシンのショートカットキーを有効化するにはCtrl+Alt+LEFT arrow
    - 全画面表示を解除するには、まずCtrl+Alt+Left arrowを押してからCtrl+Alt+Breakを押す。