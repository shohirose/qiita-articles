<!--
title:   Visual Studio CodeでIntel oneAPIを使った開発をするための設定方法
tags:    C,C++,Fortran,VSCode,intel
id:      67112fb093ce580ad307
private: false
-->
# はじめに

Visual Studio CodeでIntel oneAPIを使った開発をするための設定方法を紹介します。

参考：[Using Visual Studio Code with Intel® oneAPI Toolkits User Guide](https://www.intel.com/content/www/us/en/develop/documentation/using-vs-code-with-intel-oneapi/top/ssh-development-top/ssh-wsl.html)

# 拡張機能のインストール

MarketplaceからIntel oneAPIのための拡張機能「Extension Pack for Intel(R) oneAPI Toolkits」をインストールします。

# WSLを使う場合

常にIntel oneAPIを使って開発を行う場合は、`~/.profile`または`~/.bash_profile`に以下の行を加えて、常に環境設定が行われるようにします。

```bash
. /opt/intel/oneapi/setvars.sh &> /dev/null
```

そうでない場合は、前述の拡張機能をインストールすると、「Environment Configurator for Intel(R) oneAPI Toolkits」という拡張機能がインストールされるので、VSCodeを開いた後に以下の手順でIntel oneAPIの環境設定を有効にします。

1. `Ctrl+Shift+P`でコマンドパレットを開く。
2. 「Intel oneAPI」と入力する。
3. 「Intel oneAPI: Initialize default environment variables.」を選択する。

新しいターミナルやプロセスから環境設定が反映されます。

# CMakeとの連携

まずVisual Studio Codeの拡張機能「CMake Tools」をインストールします。そのあと、以下の2つの方法でIntelコンパイラを使ってコンパイルするように設定できます。

## settings.jsonで設定する方法

プロジェクトディレクトリの`.vscode/settings.json`にて、プロジェクトで常にIntel Compiler ClassicまたはIntel Compilerを使うように環境変数として設定します。Intel Compiler Classicの場合は

```json:settings.json
{
    "cmake.configureEnvironment": {
        "C": "/opt/intel/oneapi/compiler/latest/linux/bin/intel64/icc",
        "CXX": "/opt/intel/oneapi/compiler/latest/linux/bin/intel64/icpc",
        "FC": "/opt/intel/oneapi/compiler/latest/linux/bin/intel64/ifort"
    }
}
```

Intel Compilerの場合は

```json:settings.json
{
    "cmake.configureEnvironment": {
        "C": "/opt/intel/oneapi/compiler/latest/linux/bin/icx",
        "CXX": "/opt/intel/oneapi/compiler/latest/linux/bin/icpx",
        "FC": "/opt/intel/oneapi/compiler/latest/linux/bin/ifx"
    }
}
```

この方法ではコンパイラを切り替えることができないので、次に紹介するCMake Kitsを使った方法をおすすめします。

## CMake Kitsを設定する方法

プロジェクトディレクトリの`.vscode/cmake-kits.json`にて、下記の様にIntel Compilerの設定を行います(パスはWSLの例)。

```json:cmake-kits.json
[
    {
        "name": "Intel Compiler Classic",
        "compilers": {
            "C": "/opt/intel/oneapi/compiler/latest/linux/bin/intel64/icc",
            "CXX": "/opt/intel/oneapi/compiler/latest/linux/bin/intel64/icpc",
            "Fortran": "/opt/intel/oneapi/compiler/latest/linux/bin/intel64/ifort"
        }
    },
    {
        "name": "Intel Compiler",
        "compilers": {
            "C": "/opt/intel/oneapi/compiler/latest/linux/bin/icx",
            "CXX": "/opt/intel/oneapi/compiler/latest/linux/bin/icpx",
            "Fortran": "/opt/intel/oneapi/compiler/latest/linux/bin/ifx"
        }
    }
]
```

参考：[CMake Tools | CMake Kits](https://vector-of-bool.github.io/docs/vscode-cmake-tools/kits.html)

設定した後は、コマンドパレットを開き、「CMake: Select a kit」を選択し、上記のいずれかのKitを選択してください。その後ConfigureとBuildを行うと、選択したコンパイラーでコンパイルされます。