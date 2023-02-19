<!--
title:   OpenFOAMおよびFoam-extendに実装されているスマートポインタ（autoPtr, tmp）
tags:    OpenFOAM,foam-extend,smartpointer
id:      7715aa6c4b6f01db9df6
private: false
-->
OpenFOAM用の自作クラスを作る際は、基本的にOpenFOAMが提供するスマートポインタ（autoPtr, tmp）の使用が推奨されています。

# autoPtr

通常のスマートポインターです。

* [OpenFOAM C++ Documentation : autoPtr class](http://foam.sourceforge.net/docs/cpp/a00084.html)
* [autoPtr.H](http://foam.sourceforge.net/docs/cpp/a08392_source.html)
* [autoPtrI.H](http://foam.sourceforge.net/docs/cpp/a08393_source.html)

[OpenFOAM wiki: Understanding autoPrt](http://www.cfd-online.com/Forums/openfoam-programming-development/104220-understanding-autoptr.html)

# tmp

各種Scheme・Field用のスマートポインタです。

* [OpenFOAM C++ Documentation : tmp class](http://foam.sourceforge.net/docs/cpp/a02607.html)
* [tmp.H](http://foam.sourceforge.net/docs/cpp/a08395_source.html)
* [tmpI.H](http://foam.sourceforge.net/docs/cpp/a08396_source.html)

各種Scheme・Fieldは、メモリリークを防ぐためのrefCount Classを継承して実装されています。tmp Classを使ってScheme・Field用のメモリを動的に確保するポインタを作成すると、そのポインタのデストラクタが呼ばれた際に、ポインタが確保したメモリを開放していいか自動的に判断してくれます。

[OpenFOAM guide / tmp](http://openfoamwiki.net/index.php/OpenFOAM_guide/tmp)
[PENGUINITIS - autoPtrとtmp](http://www.geocities.jp/penguinitis2002/study/OpenFOAM/tankentai/19-autoPtr_and_tmp.html)