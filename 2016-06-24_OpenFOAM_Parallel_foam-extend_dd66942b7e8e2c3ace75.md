<!--
title:   OpenFOAMが提供する並列計算用クラス
tags:    OpenFOAM,Parallel,foam-extend,並列計算
id:      dd66942b7e8e2c3ace75
private: false
-->
# Pstream

## 参考リンク

[OpenFOAM C++ Documentation : Pstream](http://www.openfoam.com/documentation/cpp-guide/html/a02100.html)

## メンバー関数

- Pstream::nProcs() : プロセッサー数
- Pstream::myProcNo() : 自身のプロセッサーID
- Pstream::blocking, scheduled, nonBlocking : プロセッサー間のコミュニケーションの方法
- Pstream::parRun() : 並列計算ならばtrueを返す。

など

# OPstreamとIPstream

## 参考リンク

[OpenFOAM C++ Documentation : OPstream](http://www.openfoam.com/documentation/cpp-guide/html/a01785.html)
[OpenFOAM C++ Documentation : IPstream](http://www.openfoam.com/documentation/cpp-guide/html/a01240.html)

## メンバー関数

### コンストラクタ

- OPstream(const commsType, const int toProcNo, const label bufSize, ...)
- IPstream(const commsType, const int fromProcNo, const label bufSize, ...)

### 使い方の例

OpenFOAMをインストールしていればapplication/test/parallelディレクトリにテストコードがありますのでそちらを参照してください。

[github: OpenFOAM-2.1.x/applications/test/parallel/Test-parallel.C](https://github.com/OpenFOAM/OpenFOAM-2.1.x/blob/master/applications/test/parallel/Test-parallel.C)

例えばこんな使い方。

```cpp
labelList sizes(Pstream::nProcs(), 0);             // プロセッサー数と同サイズのlabelListを作成し0で初期化
sizes[Pstream::myProcNo()] = Pstream::myProcNo();  // 自身のプロセッサー番号をプロセッサー番号と同じ番地に代入

if (Pstream::parRun())
{
    // データ送信
    for (label procI = 0; procI < Pstream::nProcs(); procI++)
    {
        if (procI != Pstream::myProcNo()
        {
            OPstream toProc(Pstream::blocking, procI);
            toProc << sizes[Pstream::myProcNo()];   // 自身のデータを他のプロセッサーに送る
        }
    }


    // データ受信
    for (label procI = 0; procI < Pstream::nProcs(); procI++)
    {
        if (procI != Pstream::myProcNo()
        {
            IPstream fromProc(Pstream::blocking, procI);
            fromProc >> sizes[procI];              // 他のプロセッサーから送られてきたデータを受信
        }
    }
}
```

# Reduce Operation

## 参考リンク

[OpenFOAM C++ Documentation : PstreamReduceOps](http://www.openfoam.com/documentation/cpp-guide/html/a08023.html)
[OpenFOAM C++ Documentation : PstreamCombineReduceOps](http://www.openfoam.com/documentation/cpp-guide/html/a08246_source.html)

- void reduce(T& value, const BinaryOp& bop, ...)
- void returnReduce(T& value, const BinaryOp& bop, ...)
- void combineReduce(T& value, const CombineOp& cop, ...)


など。

BinaryOpはOpenFOAMが用意している並列計算用のオペレーションクラス
[OpenFOAM C++ Documentation : ops.H](http://www.openfoam.com/documentation/cpp-guide/html/a09141.html)
を使用する。

- sumOp\<T>()
- maxOp\<T>()
- minOp\<T>()
- andEqOp\<T>()

などが用意されている。


combineReduceはList専用のreduce関数で、combineOpはUPstream::listEq()のみ使用できる。List<List<T> >型オブジェクトを共有するために使用する。ただしfoam-extendには用意されていない。
[OpenFOAM C++ Documentation : UPstream.H](http://www.openfoam.com/documentation/cpp-guide/html/a08255_source.html)


# syncTools

## 参考リンク

[OpenFOAM C++ Documentation: syncTools Class Reference](http://cpp.openfoam.org/v4/a02631.html)

## 説明

syncToolsは各プロセッサーのメッシュのpoint, edge, face, cellのうち、processorPolyPatch上のものを同期させるユーティリティーツールです。