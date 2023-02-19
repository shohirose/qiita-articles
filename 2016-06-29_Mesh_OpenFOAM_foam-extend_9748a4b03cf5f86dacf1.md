<!--
title:   OpenFOAMのメッシュ記述方法の説明
tags:    Mesh,OpenFOAM,foam-extend
id:      9748a4b03cf5f86dacf1
private: false
-->
# 参考リンク

[OpenFOAM User Guide: 5.1 Mesh description](http://cfd.direct/openfoam/user-guide/mesh-description/)
[OpenFOAM 空間の離散化と係数行列の取り扱い by Fumiya Nozaki](http://www.slideshare.net/fumiyanozaki96/openfoam-32087641)
[Dynamic Mesh in OpenFOAM by Fumiya Nozaki](http://www.slideshare.net/fumiyanozaki96/openfoam)
[The OpenFOAM Technology Primer （本）](http://www.sourceflux.de/book/)

# 基本構造

OpenFOAM User Guideに図とともにわかりやすく記載されています。要約しますと、OpenFOAMのメッシュを構成する基本要素は

- Points（点）
- Faces（面）
- Cells（セル）

の3つで、それぞれ次のように定義され、かつ条件が設けられています。

## Points（点）

点の座標はベクトル（`Vector`)で定義されます。各点には番号が振られ（`label`）、まとめてリスト（`labelList`）として管理されます。チュートリアルの各ケースの`constant/polyMesh/points`を開いて見てください。

```cpp:points
// 点の数
624
// 点のリスト
// 点番号  (x y z)
(
0 (0 0 0)
1 (0.5 0.1 0.1)
2 (0.5 -0.1 1.0)
...
)
```

## Faces（面）

各面は面を構成する点のリスト（`labelList`）で定義されます。点は必ず面の周りを一定の向きで周るように順番づけられており、面の法線ベクトルの正の方向は、面に向かって点の順番が反時計回りならば自分の方向を正、時計回りならば自分とは反対方向を正として定義されます。

面にはinternal facesとboundary facesがあり、internal facesは必ず隣り合う2つのセルを持ちますが、boundary facesは片側のみにセルがあります。OpenFOAMでは、faceを共有する2つのセルのうちセル番号の小さいほうをowner cell、セル番号の大きい方をneighbour cellと定義しています。boundary facesは、owner cellのみを持つ面とも言い換えられます。

面は`constant/polyMesh/faces`において以下のように点の番号のリストとして表されています。面のリストの上から順に面番号（labelList）が0, 1, 2, ...と振られます。

```cpp:constant/polyMesh/faces
// 面の数
500
// 面のリスト
// 面を構成する点の数 (点番号リスト = labelList)
(
3 (0 1 2)     // face label = 0
3 (1 3 5)     // face label = 1
3 (1 2 4)     // face label = 2
4 (0 6 5 2)   // face label = 3
...
)
```

一方、境界について表している`constant/polyMesh/boundary`を見ると

```cpp:constant/polyMesh/boundary
// patchの数
8
(
    // patchの名前
    zPos
    {
        type        empty;    // patch type
        nFaces      42;       // patchを構成する面の数
        startFace   368;      // 面のリストにおけるpatchが始まる面番号
    }
    zNeg
    {
    ...
    }
    ...
)
```

のようになっているはずです。OpenFOAMでは、面のリストは必ずinternal faces → boundary facesの順に面の番号が並べられます。したがって、`constant/polyMesh/boundary`と`constant/polyMesh/faces`を比較すれば

```cpp:constant/polyMesh/faces
// 面の数
500
// 面のリスト
// 面を構成する点の数 (点番号リスト = labelList)
(
// internal faces
3 (0 1 2)     // face label = 0
3 (1 3 5)     // face label = 1
3 (1 2 4)     // face label = 2
4 (0 6 5 2)   // face label = 3
...
// boundary faces
//- zPos
3 (298 345 103)  // face label = 368
...
//- zNeg
3 (387 408 299)  // face label = 410 (= 368 + 42)
...
)
```

のように、internal facesとboundary faces（あるいは各patchを構成する面）を読み取ることができます。

## Cells（セル）

セルは面のリスト（`faceList`）で定義されます。また次の性質を満たしていなければなりません。

- Contiguous: セルは計算領域を隙間なく埋め、かつ重なってはならない。
- Convex: セルは凸であり、セルの中心はセルの内部になければならない。
- Closed: セルは幾何学的にもトポロジー的にも「閉じて」いなければならない。

前節で述べたように、セルはownerとneighbourに分けられ、それぞれ`constant/polyMesh/owner`および`constant/polyMesh/neighbour`に保存されています。

```cpp:constant/polyMesh/owner
// owner cellを持つ面の数
500
(
3   // face 0のowner cell番号
1   // face 1のowner cell番号
...
)
```

```cpp:constant/polyMesh/neighbour
// neighbour cellを持つ面の数
368  // boundary face listの最初の面の番号と等しい
(
47   // face 0のneighbour cell番号
49   // face 1のneighbour cell番号
...
)
```