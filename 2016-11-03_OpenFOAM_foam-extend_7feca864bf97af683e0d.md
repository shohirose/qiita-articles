<!--
title:   OpenFOAMにおけるファイル・ディレクトリの操作
tags:    OpenFOAM,foam-extend,ファイル操作
id:      7feca864bf97af683e0d
private: false
-->
# ファイル操作関数の定義場所

OpenFOAMでは、ファイル操作関連の関数はOSspecific.HおよびPOSIX.Cで定義されています。
[OpenFOAM C++ Documentation: OSspecific.H](http://cpp.openfoam.org/v4/a08523_source.html)
[OpenFOAM C++ Documentation: POSIX.C](http://cpp.openfoam.org/v4/a09412_source.html)

例えば
- `home()`
- `cwd()`
- `exists(const fileName&, const bool)`
- `isDir(const fileName&)`
- `isFile(const fileName&, const bool)`
などが定義されています。

# 使用例

例えば、あるGeometricFieldの初期化において、そのフィールドに対応するファイルが時間ディレクトリ内に存在する場合にファイルを読み取って初期化するするには

```cpp
if (isFile(runTime.timePath()/"sigma"))
{
    sigmaPtr_.set
    (
        new volSymmTensorField
        (
            IOobject
            (
                "sigma",
                runTime.timeName(),
                mesh,
                IOobject::MUST_READ,
                IOobject::AUTO_WRITE
            ),
            mesh
        )
    );
}
```