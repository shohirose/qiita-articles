<!--
title:   BoostをインストールするときにPython関連でエラー
tags:    C++,Python,boost
id:      caa2e7a7553adfbe64d5
private: false
-->
# エラー内容

最新のBoostライブラリをインストールしようとしたら、

> fatal error: pyconfig.h: No such file or directory

というエラーメッセージが表示され、Python関連のターゲットのビルドが失敗してしまいました。
その解決法の備忘録。

# 対策

`pyconfig.h`へのパスを見つけられなかったのが原因です。`pyconfig.h`が存在しないか、通常の場所に置かれていません。`pyconfig.h`が存在しない場合は、例えばパッケージマネージャで`python-devel`や`python3-devel`などがインストールされていないことが考えられます。`pyconfig.h`が存在するにもかかわらず見つけられない場合、解決方法は

1. `project-config.jam`へ`pyconfig.h`のパスを指定する。
2. `user-config.jam`を作成して`b2`の`--user-config`オプションに渡す。

の2つです。

## `project-config.jam`へ`pyconfig.h`のパスを追加する

- [boost 1.66 - Ubuntu 16.04 - Running b2, g++ gcc cannot find pyconfig.h #289](https://github.com/boostorg/build/issues/289)

解凍したBoostライブラリのルートディレクトリにある`project-config.jam`のpython部分を以下のように修正します。私の場合はanacondaを使っていたので以下のようになりましたが、適宜自分の場合に置き換えてpythonのバージョンやパスを加えててください。

```:変更前
if ! [ python.configured ]
{
    using python : 3.6 : /home/sho/anaconda3 ;
}
```

```:変更後
if ! [ python.configured ]
{
    using python : 3.6 : /home/sho/anaconda3 : /home/sho/anaconda3/include/python3.6m ;
}
```

## `user-config.jam`を作成して`b2`の`--user-config`オプションに渡す

解凍したBoostライブラリのルートディレクトリの中の`build/example/user-config.jam`をルートディレクトリ直下にコピーして、以下の内容をファイルの最後等に追加してください。

```user-config.jam
using python : 3.6 : /home/sho/anaconda3 : /home/sho/anaconda3/include/python3.6m ;
```

そしてルートディレクトリで`b2`を実行する際に

```console
$ ./b2 --user-config=user-config.jam install
```

と`user-config.jam`を指定すればうまくいくはずです。