<!--
title:   OpenFOAMが提供する一次元配列（UList, SubList, List, DynamicList, Xfer）
tags:    OpenFOAM,array,container,foam-extend
id:      5d48751c2a1bb565422f
private: false
-->
# UList

## 参考リンク

[OpenFOAM C++ Documentation : UList\<T>](http://www.openfoam.com/documentation/cpp-guide/html/a02840.html)
[OpenFOAM C++ Documentation : UList.H](http://www.openfoam.com/documentation/cpp-guide/html/a07891_source.html)

## 説明

Unallocated listの略で、メモリはオブジェクト生成時や実行時には割り当てられず、コンストラクタに対してアドレスを引数として与える必要があります。既存の一次元配列を参照する場合に便利なクラスです。

## メンバ関数

- `T& first()`
- `T& last()`
- `T& operator[]`
- `iterator begin()`
- `iterator end()`
- `label size()`
- `bool empty()`

など

# SubList

## 参考リンク

[OpenFOAM C++ Documentation : SubList\<T>](http://www.openfoam.com/documentation/cpp-guide/html/a02564.html)
[OpenFOAM C++ Documentation : SubList.H](http://www.openfoam.com/documentation/cpp-guide/html/a07885_source.html)

## 説明

UListを継承して作成された一次元配列の一部分を示すためのクラス。OpenFOAMには、list, containerの要素全てに対して指定された操作を実行するfor文

- `forAll (list, i)`
- `forAllReverse (list, i)`
- `forAllIter (ContainerType, container, iter)`
- `forAllConstIter (ContainerType, container, iter)`

がUlist.H内で定義されており、基本的にfor文の代わりにこれらを使います。
[OpenFOAM C++ Documentation : UList.H](http://www.openfoam.com/documentation/cpp-guide/html/a07891_source.html)

Listの一部に対してのみ操作を実行したいときは、for文を使って要素番号の始点と終点を指定するのではなく、ListからSubListを作成し、上記forAll等を使って操作を行うべきでしょう。つまり、`labelList`型オブジェクトである`list`があったとして、その要素番号`n1`から`n2`まである操作を行いたい場合、

```cpp
for (label i = n1; i <= n2; i++)
{
    list[i] = ...
}
```

とするのは不適切で、

```cpp
label subsize = n2 - n1 + 1;
SubList<label> sublist(list, subsize, n1);
forAll (sublist, i)
{
    sublist[i] = ...
}
```

が正しいと考えられます。

# List

## 参考リンク

[OpenFOAM C++ Documentation : List\<T>](http://www.openfoam.com/documentation/cpp-guide/html/a01423.html)
[OpenFOAM C++ Documentation : List.H](http://www.openfoam.com/documentation/cpp-guide/html/a07860_source.html)

## 説明

オブジェクト生成時にメモリが確保される可変長の一次元配列。

## メンバ関数

- `void append(const T&)`
- `void resize(const label)`
- `void clear()`
- `transfer(List<T> &)`

など

## typedefされた種々のList

下記以外にも`faceList`など数多くあります。

### labelList

[OpenFOAM C++ Documentation : labelList.H](http://www.openfoam.com/documentation/cpp-guide/html/a09119_source.html)

```cpp
typedef List<label> labelList;
typedef List<labelList> labelListList;
typedef List<labelListList> labelListListList;
```

### boolList

[OpenFOAM C++ Documentation : boolList.H](http://www.openfoam.com/documentation/cpp-guide/html/a09037_source.html)

```cpp
typedef UList<bool> boolUList;
typedef List<bool> boolList;
typedef List<List<bool>> boolListList;
```

# DynamicList

## 参考リンク

[OpenFOAM C++ Documentation : DynamicList](http://www.openfoam.com/documentation/cpp-guide/html/a00640.html)

## 説明

List\<T>を継承して作成された可変長の一次元配列。サイズ変更のための演算として、加減乗除が定義されています。

# Xfer

## 参考リンク

[OpenFOAM C++ Documentation : Xfer](http://www.openfoam.com/documentation/cpp-guide/html/a02994.html)
[OpenFOAM C++ Documentation : Xfer.H](http://www.openfoam.com/documentation/cpp-guide/html/a08677_source.html)

## 説明

（Xfer.Hの説明文の日本語訳）
>T型オブジェクトのコピーまたは所有権を移動するための簡単なクラス。
オブジェクトがコピーされるのか、あるいは所有権を移動されるのかはXferオブジェクトの生成時に決められるため、生成したXferオブジェクトは無条件に所有権を移動することが可能となる。これによって他のクラスにおいてコピーおよび所有権移動のそれぞれの場合においてコンストラクタやメソッドを定義する必要がなくなる。

XferオブジェクトをT型オブジェクトから生成するとき、コピーなのかまたは所有権移動なのかによってT型クラスに定義されている代入演算子`=`または移動用メンバ関数`transfer()`がそれぞれ呼び出され、コピーまたは所有権移動が行われます。

## 関数

- `Foam::Xfer<T> Foam::xferCopy(const T& t)`
- `Foam::Xfer<T> Foam::xferMove(T& t)`
- `Foam::Xfer<T> Foam::xferTmp(Foam::tmp<T>& tt)`
- `Foam::Xfer<To> Foam::xferCopyTo(const From& t)`
- `Foam::Xfer<To> Foam::xferMoveTo(From& t)`

# 様々なListに対する操作を定義した関数

[OpenFOAM C++ Documentation : ListOps.H](http://www.openfoam.com/documentation/cpp-guide/html/a07867_source.html)