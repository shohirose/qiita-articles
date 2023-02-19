<!--
title:   CMake 3.20からIntel oneAPIがサポートされた件
tags:    C,C++,CMake,Fortran,oneAPI
id:      6d2423f532a0d273b04f
private: false
-->
# 要約

- CMake 3.20.0以降で、Intel oneAPIのC/C++/Fortranコンパイラがサポートされる（一部OS除く）。
  - Intel oneAPI 2021.1以降の新世代C/C++コンパイラ(`icx`/`icpx`)は、Windows/Linux両方でサポートされている。
  - Intel oneAPI 2021.1以降の新世代Fortranコンパイラ(`ifx`)は、現時点でLinuxのみサポートされている。
- コンパイラID
  - `icx`/`icpx`/`ifx`：`IntelLLVM`
  - `icc`/`icpc`/`ifort`：`Intel`（変更なし）
- CMake 3.20.2でIntel oneAPIに係るバグ修正があったので、CMake 3.20.2以降、およびIntel oneAPI 2021.2以降を使うこと（両方共2021年5月13日現在の最新版）。

# 参考

[CMake 3.20 Release Notes](https://cmake.org/cmake/help/latest/release/3.20.html#id3)