<!--
title:   OpenFOAMをデバッグモードでコンパイルする（Ubuntu 14.04 LTS）
tags:    OpenFOAM,debug,foam-extend
id:      4087799623d241ab4ada
private: false
-->
# 参考リンク

[OpenFOAM wiki : How to debugging](https://openfoamwiki.net/index.php/HowTo_debugging)
[Debugging OpenFOAM implementation](http://www.tfd.chalmers.se/~hani/kurser/OS_CFD/debugging.pdf)
[Debugging OpenFOAM implementation with GDB](http://www.tfd.chalmers.se/~hani/kurser/OS_CFD_2010/debugging.pdf)

# 説明

## コンパイル方法

OpenFOAM wikiの手順に従えば簡単にOpenFOAMをDebugモードでコンパイルできます。簡単に手順を書くと

1. wmake用の環境変数WM_COMPILE_OPTIONをOptからDebugに変更する。

    `console
export WM_COMPILE_OPTION=Debug
source ~/.profile
`

    OpenFOAMの環境変数は$WM_PROJECT_DIR/etc/bashrcに設定されていますので、etc/bashrcを直接変更すれば以後その設定がデフォルトとなります。

    `bash:$WM_PROJECT_DIR/etc/bashrc
# WM_COMPILE_OPTION = Opt | Debug | Prof
: ${WM_COMPILE_OPTION:=Opt}; export WM_COMPILE_OPTION    //  <--このOptをDebugに書き換える
`

3. $WM_PROJECT_DIR/ThirdParty/AllMakeを実行する。

    `console
cd $WM_PROJECT_DIR/ThirdParty/
./AllMake
`

5. $WM_PROJECT_DIR/Allwmakeを実行する。

    `console
cd $WM_PROJECT_DIR
./Allwmake
`

$WM_PROJECT_DIRはOpenFOAMがインストールされているディレクトリで、OpenFOAM wikiの手順に従ってインストールしていれば、環境変数として使えるようになっています。

```console
printenv WM_PROJECT_DIR
```

とターミナルで実行すれば確認できます。もし使えなかったら、bashを使用している場合は`$HOME/.bashrc`を開き、

```shell:$HOME/.bashrc
source $HOME/foam/foam-extend-3.2/etc/bashrc
```

の一行があるかどうか確認してください。（foam-extend-3.2をインストールしている場合を書いています。他のバージョンの場合はetcより上のパスが異なるだけです。）OpenFOAM wikiのインストール手順に沿っていればこの一行が入っており、OpenFOAMの環境変数が使えるようになっています。


## デバッグ方法

まず$WM_COMPILE_OPTION=Debugとなっていることを確認してください。デバッグ対象であるアプリケーションについても、念のため、デバッグモードでコンパイルされた方の実行ファイルが呼び出されるか、次のコマンドで確認してください。

```console
which <application>
```

例えばicoFoamの場合は

```console
which icoFoam
```

となります。linux64GccDPOptではなくlinux64GccDPDebugディレクトリの実行ファイルが表示されれば問題ありません。

### 非並列計算の場合

ケースディレクトリにおいて通常通りの手順で実行すると、デバッグモードでコンパイルされたアプリケーションが呼ばれ計算が行われます。coreファイルを出力したい場合はターミナルで

```console
ulimit -c unlimited
```

と打ち込んでからアプリケーションを実行してください。

### 並列計算の場合

gdbをデバッガーとして使う場合は[OpenFOAM wiki : How to debugging, 4.2.1 mpirunDebug](https://openfoamwiki.net/index.php/HowTo_debugging)からシェルスクリプト_mpirunDebug_をダウンロードして解凍し、ケースのルートフォルダに保存してください

```console
cd <caseDir>
wget "https://openfoamwiki.net/images/2/25/MpirunDebug.tar.gz"
tar -xzvf MpirunDebug.tar.gz
```

ケースのルートフォルダで

```console
mpirunDebug -np <nProcess> <application> -parallel
```

とすればデバッグモードで並列計算が始まります。ただし、上記mpirunDebugを使用するとプログラムが自動的に開始されbreakpointの設定ができません。breakpointの設定をしたい場合は下記のようにmpirunへ引数を付けて実行してください。（ただしOpenMPI・Linux等の環境によって変わりますのであくまで参考ということで）

```console
mpirun -np <nProcess> xterm -e gdb --args <application> -parallel
```

## DebugSwitchの設定方法

OpenFOAMの各クラスにはデバッグに必要な情報を出力するための_DebugSwitch_というものが設定されています。例えばfunctionObjectsの一つであるfieldAverageのソースコードを見てください。
[OpenFOAM C++ Documentation: fieldAverage.C](http://www.openfoam.com/documentation/cpp-guide/html/a09671_source.html)
[OpenFOAM C++ Documentation: fieldAverage.H](http://www.openfoam.com/documentation/cpp-guide/html/a09672_source.html)

```cpp:fieldAverage.C
...
32 // * * * * * * * * * * * * * * Static Data Members * * * * * * * * * * * * * //
33
34 namespace Foam
35 {
36     defineTypeNameAndDebug(fieldAverage, 0);
37 }
...
```

```cpp:fieldAverage.H
...
285 public:
286
287     //- Runtime type information
288     TypeName("fieldAverage");
...
```

`defineTypeNameAndDebug(Type, DebugSwitch)`はマクロ定義で、
[OpenFOAM C++ Documentation: className.H](http://www.openfoam.com/documentation/cpp-guide/html/a08308_source.html)
[OpenFOAM C++ Documentation: defineDebugSwitch.H](http://www.openfoam.com/documentation/cpp-guide/html/a08620_source.html)
を参照すると

```cpp:className.H
86 // * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //
87 // Definitions (without debug information)
88
89 //- Define the typeName, with alternative lookup as \a Name
90 #define defineTypeNameWithName(Type, Name) \
91     const ::Foam::word Type::typeName(Name)
92
93 //- Define the typeName
94 #define defineTypeName(Type) \
95     defineTypeNameWithName(Type, Type::typeName_())
...
115 // * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //
116 // Definitions (with debug information)
117
118 //- Define the typeName and debug information
119 #define defineTypeNameAndDebug(Type, DebugSwitch) \
120     defineTypeName(Type); \
121     defineDebugSwitch(Type, DebugSwitch)
...
```

```cpp:defineDebugSwitch.H
81 //- Define the debug information, lookup as \a Name
82 #define registerDebugSwitchWithName(Type,Tag,Name) \
83     class add##Tag##ToDebug \
84     : \
85     public ::Foam::simpleRegIOobject \
86     { \
87     public: \
88         add##Tag##ToDebug(const char* name) \
89         : \
90         ::Foam::simpleRegIOobject(Foam::debug::addDebugObject, name) \
91         {} \
92         virtual ~add##Tag##ToDebug() \
93         {} \
94         virtual void readData(Foam::Istream& is) \
95         { \
96             Type::debug = readLabel(is); \
97         } \
98         virtual void writeData(Foam::Ostream& os) const \
99         { \
100            os << Type::debug; \
101        } \
102    }; \
103    add##Tag##ToDebug add##Tag##ToDebug_(Name)
104
105
106 //- Define the debug information, lookup as \a Name
107 #define defineDebugSwitchWithName(Type, Name, DebugSwitch) \
108     int Type::debug(::Foam::debug::debugSwitch(Name, DebugSwitch))
109
110 //- Define the debug information
111 #define defineDebugSwitch(Type, DebugSwitch) \
112     defineDebugSwitchWithName(Type, Type::typeName_(), DebugSwitch); \
113     registerDebugSwitchWithName(Type, Type, Type::typeName_())
```

となっています。つまり、_各クラスのヘッダーファイルに定義されているTypeNameの名前でDebugSwitchが作られ、デフォルトで0(=false)が与えられている_ということになります。

よってOpenFOAMのクラスでは、メンバ関数中にデバッグ情報を出力するコード（下記参照）を設けておくと、DebugSwitchのON/OFFを切り替えるだけでデバッグに必要な情報を自由に出力することができるようになっています。

```cpp
if (debug)
{
    // Output debug information
}
```

さらに、DebugSwitchを1,2,3,...とデバッグ情報レベルに応じて分けてコーディングしておけば、そのレベルに応じて出力させることが可能です。

```cpp
// Debug information level = 1
if (debug == 1)
{
    ...
}
// Debug information level = 2
if (debug == 2)
{
    ...
}
// Debug information level = 3
if (debug == 3)
{
    ...
}
...
```

（勿論、デバッグ情報の出力が必要なくなったならば、これらの不要になったif文を消去して計算速度を上げるべきでしょう。）

DebugSwitchはケースディレクトリの`system/controlDict`内に

```cpp:system/controlDict
DebugSwitches
{
    <TypeName>    <DebugSwitch>;
    ...
}
```

と書き込むことでON/OFFすることができます。例えば先ほどのfieldAverageクラスの場合ですと

```cpp:system/controlDict
DebugSwitches
{
    fieldAverage   1;
}
```

となります。あるいは、$WM_PROJECT_DIR/etc/controlDict(global controlDict dictionary)内のDebugSwitchesを変更する方法もあります。

# Tips

## Debugモードでコンパイルされなかった場合

コンパイル中に、各ライブラリの保存されているディレクトリがlinux64GccDPDebugではなくlinux64GccDPOptと表示されている場合、DebugモードでコンパイルできていないのでCtrl+cでコンパイルを中止しましょう。ターミナルで`env | grep WM`と打ち込んでwmake用の環境変数を確認してください。

もし`WM_COMPILE_OPTION=Opt`となっていたら`$WM_PROJECT_DIR/etc/bashrc`で環境変数が`WM_COMPILE_OPTION:=Debug`となっていない、または`$WM_PROJECT_DIR/etc/bashrc`が`source`されていないので、上記の手順に従って環境変数を変更→sourceしてください。

もし`WM_COMPILE_OPTION=Debug`となっているにもかかわらずDebugモードでコンパイルできなかったら、ターミナルを閉じて再起動してください。再起動したうえでもう一度`Allwmake`をするとDebugモードでコンパイルができるはずです。

## 実際に起きたエラー

### mpi.h
私は、`$WM_PROJECT_DIR/ThirdParty/AllMake`をDebugモードで実行するまえに`$WM_PROJECT_DIR/Allwmake`を実行してしまったので、Allwmake中にmpiに関する下記のエラーメッセージが表示されました。

```cpp
wmake libso foam
db/IOstreams/Pstreams/IPread.C:29:17: fatal error: mpi.h: No such file or directory
 #include "mpi.h"
```

これはDebugモードでコンパイルされたmpiライブラリがないために生じたようで、`$WM_PROJECT_DIR/ThridParty`をDebugモードでコンパイルしてからOpenFOAMをコンパイルするとエラーは生じませんでした。

### yyFlexLexer::yywrap()
下記のエラーメッセージが出たことがありました。

```cpp
/home/.../foam/foam-extend-3.2/lib/linux64GccDPDebug/libmeshTools.so: undefined reference to 'yyFlexLexer::yywrap()'
```

これはflexのバージョンが変わってしまったことが原因のようで、ライブラリ内にある"*.L"拡張子を持つファイル内のマクロを書き換えれば大丈夫です。
https://bugs.openfoam.org/view.php?id=1974
つまり

```cpp
#if YY_FLEX_SUBMINOR_VERSION < 34
```

を

`cpp
#if YY_FLEX_SUBMINOR_VERSION < 34 && YY_FLEX_MINOR_VERSION < 6
`
とするだけです。