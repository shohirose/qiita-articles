<!--
title:   VSCodeでCMakeのログが文字化けする
tags:    C++,CMake,VSCode,VisualStudio
id:      b2cad865b2962c57e025
private: false
-->
# 問題

Visual Studio 2019を更新した後、VSCodeでcmake-tools＆CMake&Visual C++でビルドしたところ、いきなり文字化けするようになりました。

```console
[build] cl : �R�}���h ���C�� warning D9025 : '/W3' ��� '/W4' ���D�悳��܂��B
```

Visual Studio側の問題かと思い調べたのですが、どうやらCMakeがUTF8を仮定している一方で、Visual C++の出力はシステムの規定のコードページ（私の場合はShift-JISでした）に従うことが理由のようです。
microsoft/vscode-cmake-tools: [output message is garbled #153](https://github.com/microsoft/vscode-cmake-tools/issues/153)

なぜ更新前に問題が起きなかったんだ…

# 解決方法

VSCodeの設定ファイルでエンコーディングを指定します。

```json:settings.json
{
    "cmake.outputLogEncoding": "shift-jis"
}
```

すると文字化けが治ります。

```console
[build] cl : コマンド ライン warning D9025 : '/W3' より '/W4' が優先されます。
```