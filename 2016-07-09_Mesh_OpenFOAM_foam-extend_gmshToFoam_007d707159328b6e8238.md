<!--
title:   OpenFOAM用メッシュをGmshで作成し変換する方法（gmshToFoam）
tags:    Mesh,OpenFOAM,foam-extend,gmshToFoam
id:      007d707159328b6e8238
private: false
-->
# リンク

- [OpenFOAM wiki: 2D Mesh Tutorial using GMSH]
(https://openfoamwiki.net/index.php/2D_Mesh_Tutorial_using_GMSH)
- [Gmsh](http://gmsh.info/)
- [OpenFOAM wiki: GmshToFoam](https://openfoamwiki.net/index.php/GmshToFoam)
- gmshToFoam2：[GmshToFoam21x.tar.gz](https://openfoamwiki.net/images/4/47/GmshToFoam21x.tar.gz)

# Gmshのインストール

Ubuntuの場合は

```console
$ sudo apt-get install gmsh
```

とすれば`gmsh-2.8.3`をインストールできます。最新版（2016年7月現在は`gmsh-2.12.0`）を得るにはGmshのホームページから直接ダウンロードしてください。

```console
$ wget "http://gmsh.info/bin/Linux/gmsh-2.12.0-Linux64.tgz"   # Gmsh-2.12.0をダウンロード
$ tar zxvf gmsh-2.12.0-Linux64.tgz                            # 解凍
$ cd gmsh-2.12.0-Linux                                        # ディレクトリを移動
$ sudo su                                                     # 管理者へ移行
$ cp bin/gmsh /usr/bin/                                       # 以下、各ファイルをコピー
$ cp -r share/doc/gmsh /usr/share/doc/
$ cp share/man/man1/gmsh.1 /usr/share/man/man1/
$ gmsh -version                                               # バージョンを確認 >> 2.12.0が返ってくる
```

2.8.3から2.12.0までの変更点は[Gmsh Version History](http://gmsh.info/doc/VERSIONS.txt)を確認してください。（2.9.2からSurfaceに組み込まれたPoint/LineをExtrudeすることが可能となっていますので、その機能が必要な場合は最新バージョンをインストールしてください。）

# 説明

Gmshの使い方については上のリンクを参照してください。gmshToFoamはGmshで作成したメッシュをOpenFOAMのメッシュ形式に変換するユーティリティーアプリケーションです。ケースのルートディレクトリにおいて

```console
$ gmshToFoam <gmshFileName>
```

と入力すればGmshのメッシュがOpenFOAM形式に変換され`constant/polyMesh/`に保存されます。例えばディレクトリ構造が以下のようになっているケースの場合

```console:gmshToFoam実行前のディレクトリ構成
case/
  |---0/
  |   |---p
  |   |---U
  |
  |---constant/
  |   |---polyMesh/
  |
  |---system/
  |   |---controlDict
  |   |---fvSolution
  |   |---fvScheme
  |
  |---preProcess/
      |---gmsh.geo
```

Gmshを使ってメッシュを作成し、gmshToFoamでOpenFOAM形式に変換するには次のようにコマンドを入力します。

```console:Gmshでメッシュを作成しOpenFOAM形式に変換
$ cd preProcess/
$ gmsh -3 gmsh.geo                  # 3Dメッシュファイルgmsh.mshが作成される
$ cd ..
$ gmshToFoam preProcess/gmsh.msh    # メッシュ変換
```

```console:gmshToFoam実行後のディレクトリ構成
case/
  |---0/
  |   |---p
  |   |---U
  |
  |---constant/
  |   |---polyMesh/
  |       |---sets/
  |       |---boundary
  |       |---points
  |       |---faces
  |       |---owner
  |       |---neighbour
  |       |---pointZones
  |       |---faceZones
  |       |---cellZones
  |
  |---system/
  |   |---controlDict
  |   |---fvSolution
  |   |---fvScheme
  |
  |---preProcess/
      |---gmsh.geo
      |---gmsh.msh
```

Gmsh内で定義されたPhysical SurfaceおよびPhysical Volumeは、定義された名前のままそれぞれ

- Physical Surface --> boundary（outer surfaceの場合）またはfaceZones（internal surfaceの場合）
- Physical Volume --> cellZones

と変換されます。ただしgmshToFoamには、Gmshで定義されたinternal surfaceの名前を認識できずfaceZone_#という名前でfaceZone内に保存するというバグがあります。これはgmshToFoam2を使うと避けることができるようですが、gmshToFoam2はfoam-extend-3.2には標準で含まれておらず、またファイル・クラス・関数の定義名がgmshToFoam作成時から変化しておりMakeファイルおよびソースコードの修正が必要です。