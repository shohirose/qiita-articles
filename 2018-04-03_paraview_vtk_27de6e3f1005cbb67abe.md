<!--
title:   VTKファイルの読み込み・書き込み
tags:    C++,paraview,vtk
id:      27de6e3f1005cbb67abe
private: false
-->
# VTKファイルの規格（format）

~~個人的な備忘録なので、かなり端折ってます。そのうち時間ができたら加筆するかもしれません。~~
非構造格子をVTKファイルに書き込む例を追記しました。

VTKファイルには大きく分けて

- Legacy規格
- XML規格

があります。詳しくは公式ページの[VTK file formats](http://www.vtk.org/VTK/img/file-formats.pdf)を読んでください。アカデミックの分野ではLegacy規格が今でもよく使われています。ただしLegacy規格は時系列データをサポートしていないため、時系列データを描画したい場合は、XML規格をPVDファイルと共に使う必要があります。

- [ParaView/Data formats: PVD File Format](https://www.paraview.org/Wiki/ParaView/Data_formats#PVD_File_Format)
- [Stack Overflow: How to set custom timestep values for a series of legacy VTK files in ParaView?
](https://stackoverflow.com/questions/33771609/how-to-set-custom-timestep-values-for-a-series-of-legacy-vtk-files-in-paraview)

データセットの幾何・トポロジーを決める型として

- STRUCTURED_POINTS
- STRUCTURED_GRID
- UNSTRUCTURED_GRID
- POLYDATA
- RECTILINEAR_GRID
- FIELD

が用意されています。数値計算の結果を可視化する場合、Structured GridとUnstructured Gridを最もよく使うと思います。



# VTKファイルの読み込み・書き込み

VTKライブラリに読み込み・書き込みするためのツールが含まれているので、自分でI/O用のツールを作る必要は全くありません。
（ネットを検索すると、なぜか自分でI/Oを書くような解説が結構ヒットしますが、__時間の無駄__です。）
最新版のビルド方法がわからなければ、[私の過去の記事](2018-02-10_vtk_ba709fb1d9b5ef7463c1.md)を参考にしてください。

↓の記事を読めば基本的な使い方がすぐ理解できます。
[VTKによるファイル読み込みと書き込み](https://qiita.com/vs4sh/items/76d81ae8f9e03bfae7cf)

より細かい使い方は

- [VTKExamples](https://lorensen.github.io/VTKExamples/site/)
- [VTK Documentation](https://www.vtk.org/doc/nightly/html/index.html)

を参照すれば大体でてきます。


## 非構造格子をVTKファイルに書き込むコードの例

VTKライブラリを使って非構造格子（Unstructured Grid）をLegacy規格のVTKファイルに書き込む例を示します。基本的な手順は下記の通りです。

1. Point配列を作成する
2. Cell配列を作成する
3. Point配列およびCell配列からGridデータを作成する
4. Point/Cellに付属するデータ配列を作成し、Gridデータへ与える
5. ファイルへ書き込む

この例では簡単のため４を省略しGridデータのみ書き込んでいます。

### C++

```cpp:VtkUnstructuredGridWriter.hpp
#ifndef VTK_UNSTRUCTURED_GRID_WRITER_H
#define VTK_UNSTRUCTURED_GRID_WRITER_H

#include <array>
#include <vector>
#include <string>

using Point = std::array<double, 3>;
using QuadCell = std::array<int, 4>;

void VtkUnstructuredGridWriter(const std::string& filename,
                               const std::string& header,
                               const std::vector<Point>& points,
                               const std::vector<QuadCell>& cells);

#endif
```

```c++:VtkUnstructuredGridWriter.cpp
#include <vtkSmartPointer.h>
#include <vtkUnstructuredGrid.h>
#include <vtkUnstructuredGridWriter.h>
#include <vtkQuad.h>
#include <vtkCellArray.h>
#include <vtkPointData.h>
#include <vtkCellData.h>
#include "VtkUnstructuredGridWriter.hpp"

void VtkUnstructuredGridWriter(const std::string& filename,
                               const std::string& header,
                               const std::vector<Point>& points,
                               const std::vector<QuadCell>& cells) {
// Point配列を作成する
auto vtk_points = vtkSmartPointer<vtkPoints>::New();
for (auto&& point : points) {
    vtk_points->InsertNextPoint(point.data());
}

// Cell配列を作成する
// この例の場合、Cell形式はVTK_QUAD
auto vtk_cells = vtkSmartPointer<vtkCellArray>::New();
for (auto&& cell : cells) {
    auto vtk_cell = vtkSmartPointer<vtkQuad>::New();
    // Cell１個当たりのPointの数が決まっていないCell形式
    //  - VTK_POLY_VERTEX
    //  - VTK_POLY_LINE
    //  - VTK_TRIANGLE_STRIP
    //  - VTK_POLYGON
    // の場合は、下のfor-loopを回す前に
    //   vtk_cell->GetPointIds()->SetNumberOfIds(npoints);
    // としてあらかじめリストサイズを確保するか、
    // SetId(i, id)の代わりにInsertNextId(id)を使う
    for (int i = 0; i < cell.size(); ++i) {
        vtk_cell->GetPointIds()->SetId(i, cell[i]);
    }
    vtk_cells->InsertNextCell(vtk_cell);
}

// Point配列とCell配列からGridを作成する
auto vtk_grid = vtkSmartPointer<vtkUnstructuredGrid>::New();
vtk_grid->SetPoints(vtk_points);
vtk_grid->SetCells(VTK_QUAD, vtk_cells);

// GridをVTKファイルに書き込む
auto writer = vtkSmartPointer<vtkUnstructuredGridWriter>::New();
writer->SetFileName(filename.c_str());  // ファイル名をセットする
writer->SetHeader(header.c_str());      // ヘッダーをセットする
writer->SetInputData(vtk_grid);         // Gridをセットする
writer->Write();
}
```

- [vtkPoints Class Reference](https://www.vtk.org/doc/nightly/html/classvtkPoints.html)
- [vtkIdList Class Reference](https://www.vtk.org/doc/nightly/html/classvtkIdList.html)
- [vtkCell Class Reference](https://www.vtk.org/doc/nightly/html/classvtkCell.html)
- [vtkUnstructuredGrid Class Reference](https://www.vtk.org/doc/nightly/html/classvtkUnstructuredGrid.html)
- [vtkUnstructuredGridWriter Class Reference](https://www.vtk.org/doc/nightly/html/classvtkUnstructuredGridWriter.html)


### Python

そのうち書くかも。

<!--
`python
import vtk

points = vtk.vtkPoints()
cells = vtk.vtkCells()
quad_cell = vtk.vtkQuad()
grid = vtk.vtkUnstructuredGrid()
`
-->

# CMakeの設定

CMakeを使ってビルドするとき、CMakeLists.txtに次のような設定を加えます。

```cmake
add_executable(myapp myapp.cpp)

# VTKライブラリを検索
find_package(VTK REQUIRED)
# VTKライブラリをリンク
target_link_libraries(mylib ${VTK_LIBRARIES})
# VTKライブラリのヘッダーファイルをインクルード
target_include_directories(mylib
  SYSTEM PRIVATE ${VTK_INCLUDE_DIRS}
  )
```

VTKライブラリをローカルディレクトリにインストールしている場合は、cmakeを実行するときにオプションを加えて実行します。

```cmake
cmake -DVTK_DIR:PATH=/home/usr/local/dir ..
```