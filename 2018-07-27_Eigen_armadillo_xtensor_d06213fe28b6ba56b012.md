<!--
title:   C++を使って数値計算をしている学部生へのアドバイス
tags:    C++,Eigen,armadillo,xtensor,数値計算
id:      d06213fe28b6ba56b012
private: false
-->
# はじめに

これは、主に大学に入ってからプログラミングを始め、今授業や研究でC++を使って行列の演算を含む数値計算をしている学部生を対象に書いています。

きっかけは、江添亮さんの[歌舞伎座.tech #8 「C++初心者会」を開催した](https://ezoeryou.github.io/blog/article/2015-05-18-kabukiza-tech-8.html)というブログの

> 大学でC++03を教わった私が、便利そうだと思ったC++11の新機能
>
> これも初心者らしい話。聞説、発表者の大学では、この2015年に大昔の化石規格であるC++03を教えて、それで学位を与えているようだ。日本の教育機関は10年ほど前からC++標準化委員会と関わらなくなっているため、もはや日本の教育機関に最新のC++規格をまともに把握している人間はいない。そのため、C++11を教育できる人間がいないのだろう。
>
> この2015年にC++03しか教育できない教育者しかいない大学というのは何なのだろうか。しかも発表者によると、教育内容には規格上の誤りが多かったという。

と

- [C++多重配列用のメモリ確保の方法（メモ）](https://qiita.com/yoggi/items/343eee82ff5c6d3f7599#_reference-4afefd13645bfadba738)
- [C++多重配列用のメモリ確保の方法（改良版、メモ）](https://qiita.com/yoggi/items/04d9aa58311f39f4864f)

を読んだこと。

# 本当にC++を使って計算する必要がありますか

C++を使った線形代数計算の話に入る前に、まず本当にC++を使う必要があるのかよく考えてください。
Python, Matlab, Julia等の、C++よりもプログラミングの難易度が低くて簡単に線形代数計算ができる言語があります。
そちらの方が幸せになれます。

C++を数値計算に利用する理由はただひとつ「Performance」 です。

> C++ doesn't give you performance. It gives you control over performance.
>
> _[CppCon 2014: Chandler Carruth "Efficiency with Algorithms, Performance with Data Structures"](https://youtu.be/fHNmRkzxHWs?t=9m50s)_

# `new`/`delete`を使った動的配列のメモリ確保・解放のことは忘れましょう

Bjarne StroustrupがC言語の拡張として開発したC++は

> 表現力と効率性の向上のために、手続き型プログラミング・データ抽象・オブジェクト指向プログラミング・ジェネリックプログラミングの複数のプログラミングパラダイムを組み合わせている
>  [プログラミング言語C++ 第4版, pp.12-13]

という特徴を持っています。C++の備える特徴を最大限に活用することで、プログラマーはリソースの確保・解放といった直接的・具体的な操作を気にせずに、数式を書くかのようにプログラムできます。

残念ながら、大学の工学部でのプログラミング教育はFortranやCを基本としており、最新のC++について学ぶことができる授業は（私の知る限り）ほとんどありません。したがって、自分で積極的にプログラミングを勉強しない限り、化石時代のプログラム

```cpp
int size = /* ... */

// メモリ確保
double* a = new double[size];
double* b = new double[size];
double* c = new double[size];

/* ... 配列の初期化 ... */

// ベクトル和：c = a + b
for (int i = 0; i < size; ++i)
  c[i] = a[i] + b[i];

// メモリ解放
delete [] a;
delete [] b;
delete [] c;
```

を書くことになります。こんなプログラムを書くくらいならPythonで

```python
import numpy as np

a = np.array([1, 2, 3, 4])
b = np.array([3, 0, -1, 2])
c = a + b
# c = [4, 2, 2, 6]
```

と書くほうが、作る方も読む方も幸せになれます。

ちなみにCppConのプレゼンにこんなの↓がありました。
[CppCon 2015: Kate Gregory “Stop Teaching C"](https://youtu.be/YnWhqhNdYyk)

# C++では自分で`new`/`delete`したら負けです

[C++ Core Guidelines, R: Resource Management](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#S-resource)を読み、最新のC++におけるリソース管理の指針について勉強しましょう。

> To avoid leaks and the complexity of manual resource management. C++'s language-enforced constructor/destructor symmetry mirrors the symmetry inherent in resource acquire/release function pairs such as fopen/fclose, lock/unlock, and new/delete. Whenever you deal with a resource that needs paired acquire/release function calls, encapsulate that resource in an object that enforces pairing for you -- acquire the resource in its constructor, and release it in its destructor.
>
> _[C++ Core Guidelines, R.1: Manage resources automatically using resource handles and RAII (Resource Acquisition Is Initialization)](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#r1-manage-resources-automatically-using-resource-handles-and-raii-resource-acquisition-is-initialization)_

以下和訳

> リークや手動での複雑なリソース管理を避けること。C++が言語レベルで強制するコンストラクタ／デストラクタの対称性は、fopen/fclose、lock/unlock、new/deleteのようなリソースの確保・解放をする関数に内在する対称性を反映している。リソース確保・解放関数の呼び出しをペアで行わなければならない場合は、常にあなたに代わって対となる操作を強制させるオブジェクトの中にリソースを隠蔽しなさい。すなわち、コンストラクタ内でリソースを確保し、デストラクタの中で解放しなさい。

つまり、配列を扱うなら、少なくとも`std::array`か`std::vector` を使えということです。

# 最初からEigen・Armadillo・xtensorなどのC++用線形代数ライブラリを使うようにする

[Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page)・[Armadillo](http://arma.sourceforge.net/)・[xtensor](https://github.com/QuantStack/xtensor)等のC++用の線形代数ライブラリを使って、自分でメモリの確保・解放をしたり、要素ごとにforループを回さずに済むようにしましょう。特にEigenはヘッダーのみのライブラリでインクルードするだけで使えるためおすすめです。

```cpp
#include <Eigen/Core>

/* ... */
// 要素の型がdoubleで、大きさを変更可能なベクトル
using Eigen::VectorXd;
int size = /* ... */

// コンストラクタが呼ばれ、自動的にベクトルa,bのリソースが確保される
VectorXd a(size);
VectorXd b(size);

/* ... ベクトルa, bの初期化 ... */

// ベクトル和：c = a + b
// 自分でforループを回す必要はない。
VectorXd c = a + b;

// ちなみに、EigenはExpression Templateという技法を使っているため、
// 上の式でautoを使うとcの型がVectorXdではなくなってしまう。

// スコープを抜ければ自動的にa,b,cのリソースは解放される
```