<!--
title:   OpenFOAMの提供するメッシュ（polyMesh, fvMesh, dynamicFvMesh, topoChangerFvMesh）
tags:    Mesh,OpenFOAM,foam-extend
id:      cb8ebc0c8e7389a6e517
private: false
-->
# polyMesh

## 参考リンク

[OpenFOAM C++ Documentation : polyMesh](http://openfoam.com/documentation/cpp-guide/html/a01995.html)
[OpenFOAM C++ Documentation : polyMesh.H](http://openfoam.com/documentation/cpp-guide/html/a08860_source.html)
[OpenFOAM Documentation Project : primitive mesh](http://www.tfd.chalmers.se/~hani/kurser/OS_CFD_2008/primitiveMeshDraftVersion.pdf)

## 説明

メッシュの基本要素を記述したクラス。OpenFOAMに実装されている全てのメッシュはこのクラスを継承して作成されています。

## メンバ関数

並列計算に必要なメッシュデータを参照するメンバ関数のほとんどはこのpolyMeshクラスで定義されています。

|メンバ関数|説明
|:--------|:----------------------|
|`faces()`|faceListを返す。|
|`faceOwners()`|face owner cellのlabelListを返す。|
|`faceNeighbour()`|face neighbour cellのlabelListを返す。|
|`boundaryMesh()`|polyBoundaryMeshを返す。|
|`bounds()`|メッシュのbounding boxを返す。|
|`globalData()`|globalMeshData（並列計算用のメッシュデータを取り扱っているクラス）を返す。|
など

# globalMeshData

## 参考リンク

[OpenFOAM C++ Documentation : globalMeshData](http://openfoam.com/documentation/cpp-guide/html/a00980.html)
[OpenFOAM C++ Documentation : globalMeshData.H](http://openfoam.com/documentation/cpp-guide/html/a08825_source.html)

## 説明

並列計算時に必要となるメッシュデータを定義しているクラス。sharedPoint, sharedEdgeとは、3つ以上のプロセッサーに共有されているpoint, edgeのことです。coupledPatchとは、並列計算をするためのメッシュ分割を行ってできたpatchで、隣接するメッシュを持つプロセッサーとデータをやり取りする必要があるpatchのことです。

# fvMesh

## 参考リンク

[OpenFOAM C++ Documentation : fvMesh](http://openfoam.com/documentation/cpp-guide/html/a00921.html)
[OpenFOAM C++ Documentation : fvMesh.H](http://openfoam.com/documentation/cpp-guide/html/a06044_source.html)
[OpenFOAM Documentation Project : primitive mesh](http://www.tfd.chalmers.se/~hani/kurser/OS_CFD_2008/primitiveMeshDraftVersion.pdf)

## 説明

polyMeshを継承して作成されている有限体積法用のメッシュを提供するクラス。OpenFOAMは基本的に有限体積法に基づいて計算を行うので、fvMeshクラスのメンバ関数に何があるのか、その役割は何かということを理解しておく必要があります。

## メンバ関数

|メンバ関数|説明
|:--------|:------------|
|`owner()`|owner cellのlabelUListを返す。|
|`neighbour()`|neighbour cellのlabelUListを返す。|
|`Sf()`|face面積に比例するfaceの法線ベクトルを返す。|
|`magSf()`|face面積を返す。|
|`C()`|cell中心座標を返す。|
|`Cf()`|face中心座標を返す。|
|`boundary()`|境界メッシュ（fvBoundaryMesh）を返す。|
など

## Tips

### faceの単位法線ベクトルの求め方

```cpp
// fvMeshオブジェクト
fvMesh fvm;
// faceの単位法線ベクトルのfieldの計算
const surfaceVectorField faceN = fvm.Sf()/fvm.magSf();
```

# dynamicFvMesh

## 参考リンク

[OpenFOAM C++ Documentation : dynamicFvMesh](http://www.openfoam.com/documentation/cpp-guide/html/a00634.html)


## 説明

メッシュが移動するような計算を行うためのクラス。

# topoChangerFvMesh

## 参考リンク

[OpenFOAM C++ Documentation : topoChangerFvMesh](http://www.openfoam.com/documentation/cpp-guide/html/a02737.html)

## 説明

メッシュが変形するような計算を行うためのクラス。