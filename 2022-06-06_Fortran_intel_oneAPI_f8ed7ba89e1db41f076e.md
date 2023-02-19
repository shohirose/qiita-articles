<!--
title:   Intel Fortran Compiler Classic & Intel Fortran Compilerのインストール方法
tags:    Fortran,intel,oneAPI
id:      f8ed7ba89e1db41f076e
private: false
-->
# はじめに

Intel Fortran Compiler ClassicおよびIntel Fortran CompilerはIntel oneAPIに含まれており、[Intel oneAPI Base Toolkit](https://www.intel.com/content/www/us/en/developer/tools/oneapi/toolkits.html#base-kit)および[Intel one API HPC Toolkit](https://www.intel.com/content/www/us/en/developer/tools/oneapi/toolkits.html#hpc-kit)をインストールすれば利用することができます。しかし、Base & HPC Toolkitには不要なライブラリやツールまで含まれているため、両方のToolkitをインストールすると20GB程度のディスク容量を消費してしまいます。これを避けるためにIntel Fortran Compiler Classic & Intel Fortran Compilerのみをインストールする方法を紹介します。

# インストール方法

Intel Fortran Compiler Classic & Intel Fortran Compilerをスタンドアローンでインストールするファイルを[Intel oneAPI standalone component installation files](https://www.intel.com/content/www/us/en/developer/articles/tool/oneapi-standalone-components.html#fortran)からダウンロードします。
Windows版はダウンロードしたexeファイルを実行して画面に従ってインストールしてください。Linux版はダウンロードしたshファイルをコマンドラインから実行します。引数をつけなければGUIモードで起動しますが、CLIモードで起動したい場合は下のようにオプションを加えてください。

```
$ sudo ./l_fortran-compiler_p_2022.1.0.134.sh -c -a
```

`-h`オプションを加えるとヘルプが表示されます。デフォルトのインストール場所`/opt/intel/`から変更したい場合などは、ヘルプを参照してオプションを追加してください。