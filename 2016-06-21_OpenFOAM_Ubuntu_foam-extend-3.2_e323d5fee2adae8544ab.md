<!--
title:   Ubuntu（14.04＆16.04LTS）へのFOAM-extend-3.2のインストール方法（ソースコードから）
tags:    OpenFOAM,Ubuntu,foam-extend-3.2,インストール
id:      e323d5fee2adae8544ab
private: false
-->
下記リンク先にインストール方法が記載されています。


* [OpenFOAMwiki: Installation/Linux/foam-extend-3.2/Ubuntu](http://openfoamwiki.net/index.php/Installation/Linux/foam-extend-3.2/Ubuntu)
* [OpenFOAMwiki: Installing from source code](http://openfoamwiki.net/index.php/Installation/Linux/foam-extend-3.2#Installing_from_source_code)

ここでは私がUbuntu 14.04 & 16.04 LTSへFOAM-extend-3.2をインストールした際に、wikiの手順以外に必要だったことを記載しています。

# FOAM-extendのインストール

## 手順

wikiの手順は下記のとおりです。

1. パッケージのアップデート
2. 必要なパッケージのインストール
3. FOAM-extendのgit repositoryをclone
4. ソースコードをコンパイル

## 環境変数設定について

OpenFOAMの環境変数設定に関して、Qt用環境変数`$QT_BIN_DIR`の設定を間違えやすいので追記しておきます。
[OpenFOAM wiki: Installation/Linux/foam-extend-3.2](https://openfoamwiki.net/index.php/Installation/Linux/foam-extend-3.2#Installing_from_source_code)
wikiのcompileの2.で`$QT_BIN_DIR`を下のように設定するように書かれています。

```console
$ export QT_BIN_DIR=/path/to/qmake_directory
$ echo "export QT_BIN_DIR=$QT_BIN_DIR" >> etc/prefs.sh
```

qmakeディレクトリは

```console
$ which qmake
```

と入力すれば得ることができます。例えば私の場合は

```console
$ /usr/bin/qmake
```

です。このとき以下のようにexportして下さい。

```console
$ export QT_BIN_DIR=/usr/bin
$ echo "export QT_BIN_DIR=$QT_BIN_DIR" >> etc/prefs.sh
```

bin以下のqmakeまで加えてしまうとparaviewのコンパイル時にエラーが起きます。

# 実際に追加でしたこと

* g++コンパイラのインストール

    `console
$ sudo apt-get install g++
`

* gitのインストール

    `console
$ sudo apt-get install git
`

* Paraviewのインストール
    qmakeの環境変数がうまく設定されていないとParaviewがインストールされません。ただしUbuntuではわざわざコンパイルしなくても下記のように入力すればインストールすることができます。

    `console
$ sudo apt-get install paraview
`

# コンパイルエラーについて

## 参照すべき場所

[OpenFOAM wiki: Installation/swak4Foam/Understanding Error Messages](https://openfoamwiki.net/index.php/Installation/swak4Foam/Understanding_Error_Messages)
[OpenFOAM User Guide: 3.2 Compiling applications & libraries](http://cfd.direct/openfoam/user-guide/compiling-applications/)

## 実際に自分が経験したエラー

### libccmio-2.6.1.tar.gzをダウンロードできない（ThirdPartyのAllMake中）

ThirdPartyディレクトリ内のパッケージのAllMake中に、libccmio-2.6.1.tar.gzをMakeファイル中に指定されたURLからダウンロードできないというエラーメッセージが表示されました。調べてみると、libccmio libraryのURLが変わったことが原因でした。`ThirdParty/AllMake.stage3`の95行目にlibccmio libraryのURLが書かれています

```sh:ThirdParty/AllMake.stage3
95     ( rpm_make -p libccmio-2.6.1 -s libccmio-2.6.1.spec -u http://portal.nersc.gov/svn/visit/tags/2.4.2/third_party/libccmio-2.6.1.tar.gz )
```

ので、以下のように元のコマンドをコメントアウトして、正しいURLで書き換えたコマンドを追加してください。

```sh:ThirdParty/AllMake.stage3
95 #   ( rpm_make -p libccmio-2.6.1 -s libccmio-2.6.1.spec -u http://portal.nersc.gov/svn/visit/tags/2.4.2/third_party/libccmio-2.6.1.tar.gz )
96     ( rpm_make -p libccmio-2.6.1 -s libccmio-2.6.1.spec -u http://portal.nersc.gov/svn/visit/trunk/third_party/libccmio-2.6.1.tar.gz )
```

### parmetis-4.0.3.tar.gzが壊れていて展開できない（ThirdPartyのAllMake中）

`ThirdParty/rpmBuild/SOURCES/`に保存されているparmetis-4.0.3.tar.gzが壊れていて展開できなかったので、[ここ](http://glaros.dtc.umn.edu/gkhome/metis/parmetis/download)から直接ダウンロードして上書きし、再度AllMakeしました。

### cannot find -liberty（foam-extend-3.2のAllwmake中）

ソースコードのコンパイラ中に

```DOS
cannot find -l<package name>
```

と表示されることがあります。これはライブラリが未インストールまたはライブラリの古いバージョンから最新バージョンへ適切にリンクが張られていないことを示しており、

```console
$ sudo apt-get install lib<package name>-dev
```

とすると解決されます。私の場合は

```console
$ cannot find -liberty
```

とエラーメッセージが表示されたので

```console
$ sudo apt-get install libiberty-dev
```

を実行しました。

## scotchDecompがコンパイルできない（foam-extend-3.2のAllwmake中）

↓のエラーメッセージが表示されscotchDecompのコンパイルができませんでした。

```console
$ Making dependency list for source file scotchDecomp/scotchDecomp.C
$ could not open file scotch.h for source file scotchDecomp/scotchDecomp.C
$ In file included from scotchDecomp/scotchDecomp.C:107:0:
$ scotchDecomp/scotchDecomp.H:38:33: fatal error: decompositionMethod.H: そのようなファイルやディレクトリはありません
$ compilation terminated.
$ make: *** [Make/linux64GccDPOpt/scotchDecomp.o] エラー 1
```

現在原因を調査中です。

## OpenMPIに係るエラー(Ubuntu 16.04 LTS)

Ubuntu 14.04 LTSから16.04 LTSへアップグレードしたのち、foam-extend-3.2をインストールしてgmshを使用したら以下のエラーメッセージが出てきました。

```
--------------------------------------------------------------------------
It looks like opal_init failed for some reason; your parallel process is
likely to abort.  There are many reasons that a parallel process can
fail during opal_init; some of which are due to configuration or
environment problems.  This failure appears to be an internal failure;
here's some additional information (which may only be relevant to an
Open MPI developer):

  opal_shmem_base_select failed
  --> Returned value -1 instead of OPAL_SUCCESS
--------------------------------------------------------------------------
--------------------------------------------------------------------------
It looks like orte_init failed for some reason; your parallel process is
likely to abort.  There are many reasons that a parallel process can
fail during orte_init; some of which are due to configuration or
environment problems.  This failure appears to be an internal failure;
here's some additional information (which may only be relevant to an
Open MPI developer):

  opal_init failed
  --> Returned value Error (-1) instead of ORTE_SUCCESS
--------------------------------------------------------------------------
--------------------------------------------------------------------------
It looks like MPI_INIT failed for some reason; your parallel process is
likely to abort.  There are many reasons that a parallel process can
fail during MPI_INIT; some of which are due to configuration or environment
problems.  This failure appears to be an internal failure; here's some
additional information (which may only be relevant to an Open MPI
developer):

  ompi_mpi_init: ompi_rte_init failed
  --> Returned "Error" (-1) instead of "Success" (0)
--------------------------------------------------------------------------
*** An error occurred in MPI_Init
*** on a NULL communicator
*** MPI_ERRORS_ARE_FATAL (processes in this communicator will now abort,
***    and potentially your MPI job)
[oasis:25561] Local abort before MPI_INIT completed successfully; not able to aggregate error messages, and not able to guarantee that all other processes were killed!
```

ネット上で調べたところ、OpenMPIのバージョンの違いによるもののようですが、よくわかりませんでした。（例えば以下のリンクを参照）
http://stackoverflow.com/questions/36156822/error-when-starting-open-mpi-in-mpi-init-via-python
http://stackoverflow.com/questions/26901663/error-when-running-openmpi-based-library

なのでfoam-extend-3.2/etc/bash内の環境変数WM_MPLIBをOPENMPIからSYSTEMOPENMPIに変更し、Ubuntuのパッケージマネジャーでインストールされているものを使用することとしました。すると今度は以下のエラーメッセージが表示されました。

```
dirname: missing operand
Try 'dirname --help' for more information.
The program 'mpicc' can be found in the following packages:
 * lam4-dev
 * libmpich-dev
 * libopenmpi-dev
Ask your administrator to install one of them
```

libopenmpi-devをインストールすればよさそうなのでインストール。
さらにgmshToFoamを使うときに下記のエラーメッセージが表示されました。

```
error while loading shared libraries: libmpi.so.1: cannot open shared object file: No such file or directory
```

これはOpenMPIのバージョンが違うことが原因らしく、正しいlibmpi.so.#へのシンボリックリンクを作成してあげればいいようです。
http://stackoverflow.com/questions/36860315/error-because-file-libmpi-so-1-missing

私は~/.local/libにリンクを作成し、そのディレクトリへのパスを追加しました。

```
cd ~/.local/lib
ln -s /usr/lib/libmpi.so.12 libmpi.so.1
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$HOME/.local/lib
```