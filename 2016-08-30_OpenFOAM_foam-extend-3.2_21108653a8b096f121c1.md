<!--
title:   OpenFOAMとfoam-extendの異なる部分のメモ（随時追加）
tags:    OpenFOAM,foam-extend-3.2
id:      21108653a8b096f121c1
private: false
-->
# 定数

OpenFOAM/foam-extendでは数値計算用定数は

```cpp
e          // M_E
pi         // M_PI
twoPi      // 2*M_PI
piByTwo    // 0.5*M_PI
```

で定義されています。注意しておきたいのは、OpenFOAMとfoam-extendで定数が所属する名前空間が異なる点です。OpenFOAMは

```cpp
Foam::constant::mathematical
```

で、foam-extend-3.2は

```cpp
Foam::mathematicalConstant
```

です。

# autoPtr

OpenFOAM/foam-extend-3.2の提供するスマートポインタであるautoPtrのコンストラクタに微妙な違いがあります。OpenFOAMのautoPtrには

```cpp
template<class T>
inline Foam::autoPtr<T>::autoPtr(const autoPtr<T>& ap, const bool reUse)
```

がありますが、foam-extend-3.2にはありません。したがって、自作クラスのコピーコンストラクタにおいてautoPtrをcloneさせたい場合、OpenFOAMでは

```cpp
Foam::myClass:myClass(const myClass& mc)
:
    myPtr_(mc.myPtr_, false)
{}
```

とすればいいのですが、foam-extend-3.2の場合はわざわざ

```cpp
Foam::myClass:myClass(const myClass& mc)
:
    myPtr_(NULL)
{
    if (mc.myPtr_.valid())
    {
        myPtr_.set(mc.myPtr_->clone().ptr());
    }
}
```

とする必要があります。

# Listのメンバ関数：append

OpenFOAMには

```cpp:OpenFOAMのListクラスのappend
void append(const T&);
void append(const UList<T>&);
void append(const UIndirectList<T>&);
```

が用意されていますが、foam-extend-3.2には

```cpp:OpenFOAMのListクラスのappend
void append(const UList<T>&);
void append(const UIndirectList<T>&);
```

しか用意されていません。したがって、Listの最後にT型オブジェクトを加えたいとき、OpenFOAMは

```cpp:OpenFOAM
label i = 1;
labelList plainList;
plainList.append(i);
```

とできるのですが、foam-extendの場合はresizeを使って

```cpp:foam-extend
label i = 1;
labelList plainList;
plainList.resize(plainList.size() + 1, i);
```

とする必要があります。ただし可変長配列として定義されたDynamicListがありますので、オブジェクト化時点でサイズが不明な場合はDynamicListを使うことをお勧めします。（DynamicListの場合は、OpenFOAMとfoam-extendの両方共に`append(const T&)`が用意されています。）

```cpp:OpenFOAM,foam-extend共通
label i = 1;
DynamicList<label> dynList;
dynList.append(i);
```

このようにDynamicListへ追加を繰り返し、最終的にサイズが決まったところでDynamicListをListに変換するというのが正しい使い方と思われます。つまり

```cpp:DynamicList-->Listへの変換（コピーではなくポインタの所有権移動）
DynamicList<label> dynList;
...                                   // 要素の追加
labelList plainList(dynList.xfer());  // List<T>へ変換
```

xferについては[Xfer\<T> Class](http://www.openfoam.com/documentation/cpp-guide/html/a03136.html)を参照してください。

# fvPatchのメンバ関数：start

OpenFOAMのfvPatchは、patchが始まる面のlabelを返すstart関数がありますが、foam-extendにはありません。OpenFOAMのfvPatch::start()はpolyPatch::start()のwrapperなので、foam-extendの場合はpolyPatchに直接アクセスする必要があります。

```cpp:OpenFOAM
const label patchI = 0;
const fvBoundaryMesh& fvBMesh = mesh.boundary();       // meshはfvMeshオブジェクト、fvBoundaryMeshを取得
const fvPatch& fvp = fvBMesh[patchI];                  // fvPatchが返ってくる
const label start = fvp.start();
```

```cpp:foam-extend
const label patchI = 0;
const polyBoundaryMesh& pBMesh = mesh.boundaryMesh();  // meshは上と同様fvMeshオブジェクト、polyBoundaryMeshを取得
const polyPatch& pp = pBMesh[patchI];                  // polyPatchが返ってくる
const label start = pp.start();
```


# fvBoundaryMeshのメンバ関数：findPatchID

OpenFOAMのfvBoundaryMeshにはpatchIDを探して返すfindPatchID関数がありますが、foam-extendにはありません。どちらもpolyBoundaryMeshにfindPatchIDがありますので、foam-extendの場合はpolyBoundaryMesh::findPatchIDを呼び出すことになります。


# fvPatchMapper

## OpenFOAMの場合

unmapped faceがあった場合、そのface IDのaddressingは

- direct = true --> -1が与えられる。
- direct = false --> labelListはゼロサイズ

となり、さらにhasUnmapped=trueとなります。したがって、fvPatchのautoMap関数内で新たにPatchに追加されたfaceの値を初期化しようとする場合、下記のようなコードとなります。

```cpp:OpenFOAM
void myFvPatchField::autoMap(fvPatchFieldMapper& m)
{
    ...
    if (!m.hasUnmapped())
    {
        return;
    }

    if (m.direct())
    {
        // direct addressingの場合
        const labelList& addr = m.directAddressing();

        forAll(addr, newFaceI)
        {
            // direct addressingの場合、unmapped faceの値は-1になっている
            if (addr[newFaceI] == -1)
            {
                ... // newFaceIに対応するfieldの値の初期化
            }
        }
    }
    else
    {
        // indirect addressingの場合
        const labelListList& addr = m.addressing();

        forAll(addr, newFaceI)
        {
             const labelList& addrI = addr[i];

             // indirect addressingの場合、unmapped faceのlistはゼロサイズになっている
             if (addrI.size() == 0)
             {
                 ... // newFaceIに対応するfieldの値の初期化
             }
        }
    }
}
```

## foam-extendの場合

unmapped faceがあった場合、そのface IDのaddressingは

- direct == true --> 0が与えられる
- direct == false --> labelListのサイズは1、labelは0が与えられる

となります。さらにfoam-extendの場合メンバ関数hasUnmapped()が用意されていません。したがって、fvPatchのautoMap関数内で新たにPatchに追加されたfaceの値を初期化しようとする場合、下記のようなコードとなります。

```cpp:foam-extend
void myFvPatchField::autoMap(fvPatchFieldMapper& m)
{
    ...
    if (m.direct())
    {
        // direct addressingの場合
        const labelList& addr = m.directAddressing();

        forAll(addr, newFaceI)
        {
            // direct addressingの場合、unmapped faceの値は0になっているので
            // face ID = 0のfaceに対して初期化を実行するか否かの判定が必要。
            // 新たなfaceは常にfvPatchリストの末尾に追加されるので
            // m.sizeBeforeMapping() == 0のとき以外はface ID = 0の値を初期化
            // する必要はない。
            if
            (
                (addr[newFaceI] == 0 && newFaceI != 0)
             || m.sizeBeforeMapping() == 0
            )
            {
                ... // newFaceIに対応するfieldの値の初期化
            }
        }
    }
    else
    {
        // indirect addressingの場合
        const labelListList& addr = m.addressing();

        forAll(addr, newFaceI)
        {
            const labelList& addrI = addr[i];

            // indirect addressingの場合もdirect addressingと同様の判定が必要
            if
            (
                (
                    addrI.size() == 1
                 && addrI[0] == 0
                 && m.sizeBeforeMapping() != 0
                )
             || m.sizeBeforeMapping() == 0
            )
            {
                ... // newFaceIに対応するfieldの値の初期化
            }
        }
    }
}
```