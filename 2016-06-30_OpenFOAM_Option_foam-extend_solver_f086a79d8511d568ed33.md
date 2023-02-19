<!--
title:   OpenFOAMのソルバーアプリケーション実行時に与えることのできるオプション
tags:    OpenFOAM,Option,foam-extend,solver
id:      f086a79d8511d568ed33
private: false
-->
# 参考リンク

[OpenFOAM® C++ Documentation: icoFoam (incompressible flow solvers)](http://www.openfoam.com/documentation/cpp-guide/html/a03356.html)
[OpenFOAM® C++ Documentation: icoFoam.C](http://www.openfoam.com/documentation/cpp-guide/html/a03356_source.html)

# 説明

一番簡単なチュートリアルにも使用されているicoFoamを使って説明します。上のicoFoam.Cのリンクを見てください。main関数の一行目で`setRootCase.H`というヘッダーファイルが読み込まれています。

```cpp:icoFoam.C
62 ...
63 #include "fvCFD.H"
64 #include "pisoControl.H"
65
66 // * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //
67
68 int main(int argc, char *argv[])
69 {
70     #include "setRootCase.H"    // 引数の読み込み
71     #include "createTime.H"
72     #include "createMesh.H"
73 ...
```

そこで
[OpenFOAM® C++ Documentation: setRootCase.H](http://www.openfoam.com/documentation/cpp-guide/html/a08439_source.html)
を見てみると下のようになっています。

```cpp:setRootCase.H
1 //
2 // setRootCase.H
3 // ~~~~~~~~~~~~~
4
5 Foam::argList args(argc, argv);
6 if (!args.checkRootCase())
7 {
8     Foam::FatalError.exit();
9 }
```

`Foam::argList`クラスのオブジェクト`args`が作られているのがわかります。この`Foam::argList`クラスの定義
[OpenFOAM® C++ Documentation: argList](http://www.openfoam.com/documentation/cpp-guide/html/a00070.html)
を見てみると、上から4分の1くらいのところにデフォルトで設定されているコマンドラインオプションが説明されています。

|オプション  |説明|
|:----------|:------------------------------|
|-case \<dir>|現在のディレクトリではなく指定されたケースディレクトリを選択する。|
|-parallel   |ケースを並列ジョブとして与える。|
|\-doc        |文書をブラウザに表示する。|
|\-srcDoc     |ソースの文書をブラウザに表示する。|
|\-help       |使用方法を表示する。|

よって例えば

```console
$ icoFoam -case test1
```

と入力するとカレントディレクトリ内にある`test1`というサブディレクトリに作られているケースを読み取ってicoFoamを実行するということになります。

-parallelは並列計算時に使用するオプションです。例えば４つのプロセッサーを使って並列計算する場合は

```console
$ mpirun -np 4 icoFoam -parallel
```

デバッグモードの場合は

```console
$ mpirunDebug -np 4 icoFoam -parallel
```

とします。

[OpenFOAM® v3.0+: New Parallel Functionality](http://www.openfoam.com/version-v3.0+/parallel.php)
[OpenFOAM® wiki: How to debugging](https://openfoamwiki.net/index.php/HowTo_debugging)