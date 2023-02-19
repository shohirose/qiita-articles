<!--
title:   ポリシーを使った3次の状態方程式の実装
tags:    C++,policy,template,数値計算
id:      190af8ae590e85deada3
private: false
-->
# はじめに

最新の論文を元にして状態方程式を用いた気体の状態変化や物性値の予測を仕事ですることになりそうなので、現在そのためのライブラリ「eoscpp」を個人的に作っています。その一部をここで紹介します。以下では簡単のため純物質の場合のみを説明します。

現在は純物質のみの場合をGitHubで公開しています。順次機能を追加していく予定です。
[shohirose/eoscpp](https://github.com/shohirose/eoscpp)

## 今後追加する予定の機能

- [x] エンタルピー計算
- [x] エントロピー計算
- [x] ギブスエネルギー計算
- [ ] 多成分系への拡張
- [x] フラッシュ計算
- [x] 沸点圧力計算

# 状態方程式とは

## 理想気体の状態方程式

熱力学において気体の温度・圧力・体積の関係を表す式は状態方程式(Equation of State, EoS)と呼ばれます。高校生で習うのが理想気体の状態方程式です。

```math
PV=NRT
```

ここで$P$は圧力、$V$は体積、$N$は物質量、$R$は気体定数、$T$は温度です。一般に、圧力が低く温度が高ければ理想気体の状態方程式に従います。

## 実在気体の状態方程式

実際には、実在気体は理想気体のように振る舞わず、理想気体の状態方程式からずれが生じます。実在気体の状態方程式では、このずれを圧縮係数$Z$として表します。

```math
Z := \frac{PV}{NRT}
```

理想気体の場合は常に$Z=1$です。様々な気体について実験的に圧縮係数が測定されていますが、実在気体の挙動を予測するためには圧縮係数を予測可能なモデルが不可欠です。


## 3次の状態方程式

3次の状態方程式(cubic EoS)は、この実在気体の物性や挙動を予測するモデルとして様々な工学分野で使用されています。最も広く使われている3次の状態方程式は、分子間力定数$a$と分子排除容積$b$の2つの定数で表されるタイプで、以下の3つのEOSが有名です。

- Van der Waals EoS (VDW EoS)[^1]
- Soave-Redlich-Kwong EoS (SRK EoS)[^2]
- Peng-Robinson EoS (PR EoS)[^3]

[^1]: van der Waals; J. D. (1873). On the Continuity of the Gaseous and Liquid States (doctoral dissertation). Universiteit Leiden.
[^2]: Soave, Giorgio (1972). "Equilibrium constants from a modified Redlich-Kwong equation of state". Chemical Engineering Science. 27 (6): 1197–1203. doi:10.1016/0009-2509(72)80096-4.
[^3]: Peng, D. Y.; Robinson, D. B. (1976). "A New Two-Constant Equation of State". Industrial and Engineering Chemistry: Fundamentals. 15: 59–64. doi:10.1021/i160057a011.

これらの状態方程式は次のように表されます。

| EoS | 状態方程式 |
|:------|:------|
|VDW | $\displaystyle P = \frac{RT}{V-Nb} - \frac{N^2 a}{V^2} $ |
|SRK | $\displaystyle P = \frac{RT}{V-Nb} - \frac{N^2 a}{V(V+Nb) } $ |
|PR  | $\displaystyle P = \frac{RT}{V-Nb} - \frac{N^2 a}{V^2+2NbV-(Nb)^2} $ |

モル体積$v:=V/N$を使うと次のように表されます。

| EoS | 状態方程式 |
|:------|:------|
|VDW | $\displaystyle P = \frac{RT}{v-b} - \frac{a}{v^2} $ |
|SRK | $\displaystyle P = \frac{RT}{v-b} - \frac{a}{v(v+b) } $ |
|PR  | $\displaystyle P = \frac{RT}{v-b} - \frac{a}{v^2+2bv-b^2} $ |

SRKとPRでは分子間力定数が温度に依存するとして、$a=a(T)=a_c \alpha(T)$とモデル化されます。ここで$a_c:=a(T_c)$、$\alpha(T)$は分子間力定数の温度依存性を補正する係数で、$\alpha(T_c)=1$となるように定義されます。$\alpha$には様々な式が提案されていますが、次式[^2]がよく使われています。

```math
\alpha(T) = \left[ 1 + m \left( 1-\sqrt{T_r} \right) \right]^2
```

$T_r:=T/T_c$は対臨界温度、$m$は分子の偏心を補正する係数です。

| EoS | $m$の計算式 |
|:------|:------|
|SRK | $ m = 0.480+1.574\omega -0.176\omega^2 $
|PR  | $ m = 0.37464+1.54226\omega - 0.26992\omega^2 $

ここで$\omega$は偏心係数です。

2つの定数で表される3次の状態方程式において、分子間力定数と分子排除容積が温度と組成のみに依存する場合、臨界点における分身間力定数と分子排除容積は臨界定数から一意に定まります。

```math
a_c = \Omega_a \frac{R^2 T_c^2}{P_c}, \quad b_c = \Omega_b \frac{R T_c}{P_c}
```

ここで$P_c$は臨界圧力、$T_c$は臨界温度です。$\Omega_a$、$\Omega_b$はEOSごとに決まっている定数です。

|EoS |$\Omega_a$|$\Omega_b$|
|:---|:--------:|:--------:|
|VDW |0.421875  |0.125     |
|SRK |0.42747   |0.08664   |
|PR  |0.45724   |0.07780   |

実在気体の状態方程式と3次の状態方程式を合わせると、圧縮係数に関する3次方程式が得られます。

|EoS | 圧縮係数に関する3次方程式 |
|:---|:----------------------|
|VDW | $Z^3-(1+B)Z^2+Az-AB=0$ |
|SRK | $Z^3-Z^2+(A-B-B^2)Z-AB=0$ |
|PR  | $Z^3+(B-1)Z^2+(A-3B^2-2B)Z-(AB-B^2-B^3)=0$ |

ここで$A$、$B$は次式で定義されます。

```math
\begin{align}
A &:= \frac{aP}{R^2T^2} = \Omega_a \frac{P_r}{T_r^2} \\
B &:= \frac{bP}{RT} = \Omega_b \frac{P_r}{T_r}
\end{align}
```

$P_r$は対臨界圧力、$T_r$は対臨界温度です。状態方程式の基礎を学びたいならKyle (1999)[^4]を読むと良いでしょう。応用についてはKontogeorgis and Folas(2010)[^5]にまとめられています。

[^4]: Kyle, B. G. 1999. Checmical and Process Thermodynamics, third edition. Pearson.
[^5]: Kontogeorgis, G. M. and Folas, G. K. 2010. Thermodynamic Models for Industrial Applications. John Wiley & Sons.

# 3次の状態方程式の実装

さて、前節で説明した3次の状態方程式をC++を使って実装していきます。ここではC++らしく、EOSごとに異なる部分をポリシーを使ってテンプレートパラメータ化し、実装します。

## 共通部分: `CubicEos`

```cpp
/// Gas constant [J/mol-K]
constexpr double gas_constant = 8.31446261815324;

/// @brief Two-parameter cubic EoS
/// @tparam T Value type
/// @tparam Eos EoS policy
/// @tparam Corrector Temperature correction for attraction parameter
template <typename T, typename Eos, typename Corrector>
class CubicEos {
 public:
  // Static functions

  /// @brief Computes attraction parameter
  /// @param[in] pc Critical pressure
  /// @param[in] tc Critical temperature
  /// @returns Attraction parameter at critical condition
  static T attraction_param(const T &pc, const T &tc) noexcept {
    return (Eos::omega_a * gas_constant * gas_constant) * tc * tc / pc;
  }

  /// @brief Computes repulsion parameter
  /// @param[in] pc Critical pressure
  /// @param[in] tc Critical temperature
  /// @returns Repulsion parameter at critical condition
  static T repulsion_param(const T &pc, const T &tc) noexcept {
    return (Eos::omega_b * gas_constant) * tc / pc;
  }

  /// @brief Computes reduced attraction parameter
  /// @param[in] pr Reduced pressure
  /// @param[in] tr Reduced temperature
  /// @returns Reduced attraction parameter
  static T reduced_attraction_param(const T &pr, const T &tr) noexcept {
    return Eos::omega_a * pr / (tr * tr);
  }

  /// @brief Computes reduced repulsion parameter
  /// @param[in] pr Reduced pressure
  /// @param[in] tr Reduced temperature
  /// @returns Reduced repulsion parameter
  static T reduced_repulsion_param(const T &pr, const T &tr) noexcept {
    return Eos::omega_b * pr / tr;
  }

  // Constructors

  /// @brief Constructs EoS.
  /// @param[in] pc Critical pressure
  /// @param[in] tc Critical temperature
  /// @param[in] corrector Corrector
  CubicEos(const T &pc, const T &tc, const Corrector& corrector)
      : pc_{pc},
        tc_{tc},
        ac_{this->attraction_param(pc, tc)},
        bc_{this->repulsion_param(pc, tc)},
        corrector_{corrector} {}

  /// @brief Computes pressure at given temperature and volume.
  /// @param[in] t Temperature
  /// @param[in] v Volume
  /// @returns Pressure
  T pressure(const T &t, const T &v) noexcept {
    const auto tr = t / tc_;
    const auto a = corrector_.alpha(tr) * ac_;
    const auto b = bc_;
    return Eos::pressure(t, v, a, b);
  }

  /// @brief Computes z-factor
  /// @param[in] p Pressure
  /// @param[in] t Temperature
  /// @returns An array of z-factors
  std::vector<T> zfactor(const T& p, const T& t) const noexcept {
    const auto pr = p / pc_;
    const auto tr = t / tc_;
    const auto a = corrector_.alpha(tr) * this->reduced_attraction_param(pr, tr);
    const auto b = this->reduced_repulsion_param(pr, tr);
    const auto p = Eos::cubic_eq(a, b);
    return real_roots(p);
  }

 private:
  T pc_;
  T tc_;
  T ac_;
  T bc_;
  Corrector corrector_;
};
```

$a$および$b$の計算は、`attraction_param`および`repulsion_param`という静的関数で行います。EOSごとに定められる定数$\Omega_a$と$\Omega_b$は、テンプレートパラメータ`Eos`において`omega_a`と`omega_b`という定数として定義されているとしています。

一方、$A$および$B$の計算は、`reduced_attraction_param`および`reduced_repulsion_param`という静的関数で行います。ただし、$\alpha$は除いて計算し、あとから$\alpha$を掛ける形にしています。

各EOSに特有の計算、すなわち

- PVTの式
- 圧縮係数の三次方程式

については、`Eos`の静的関数として実装します。分身間力定数の温度依存性を表す係数$\alpha$は、`Corrector`というテンプレートパラメータのメンバー関数で計算するようにしています。

## 三次方程式の実根を求める関数: `real_roots`

`CubicEos`の`zfactor`関数で使われている`real_roots`は、三次方程式の実根を求める関数です。

```cpp
/// @brief Computes real roots of a cubic equation.
/// @param[in] a Array of coefficients of a cubic equation
/// @returns Array of real roots of a cubic equation
///
/// The cubic equation takes the form of:
/// \f[
///   x^3 + a[0] x^2 + a[1] x + a[2] = 0.
/// \f]
template <typename T>
std::vector<T> real_roots(const std::array<T, 3>& a) noexcept {
  const auto x = roots(a);
  std::vector<T> xreal;
  xreal.reserve(3);
  using std::fabs;
  constexpr double eps = 1e-10;
  for (auto&& xi : x)
    if (fabs(xi.imag()) < eps) xreal.push_back(xi.real());
  return xreal;
}
```

また`roots`は複素数の根をカルダノの公式を使って求める関数です。

```cpp
/// @brief Computes roots of a cubic equation by using Cardano's formula.
/// @param[in] a Array of coefficients of a cubic equation
/// @returns Array of roots of a cubic equation
///
/// The cubic equation takes the form of:
/// \f[
///   x^3 + a[0] x^2 + a[1] x + a[2] = 0.
/// \f]
template <typename T>
std::array<std::complex<T>, 3> roots(const std::array<T, 3>& a) noexcept {
  const auto p = (3 * a[1] - a[0] * a[0]) / 9;
  const auto q = (27 * a[2] + a[0] * (2 * a[0] * a[0] - 9 * a[1])) / 54;
  // Discriminant of the cubic equation
  const auto disc = p * p * p + q * q;

  using std::pow;
  using std::sqrt;
  using std::complex;

  const auto s = sqrt(complex<T>(disc, 0));
  const auto u1 = pow(-q + s, 1.0 / 3.0);
  const auto u2 = pow(-q - s, 1.0 / 3.0);

  const auto sqrt3 = sqrt(3.0);
  // The primitive cube root of unity
  const auto w1 = complex<T>(-0.5, sqrt3 / 2);
  const auto w2 = complex<T>(-0.5, -sqrt3 / 2);

  // Roots based on Cardano's formula
  const auto x1 = u1 + u2 - a[0] / 3;
  const auto x2 = w1 * u1 + w2 * u2 - a[0] / 3;
  const auto x3 = w2 * u1 + w1 * u2 - a[0] / 3;

  return {x1, x2, x3};
}
```


## ポリシー：`Eos`

ポリシー`Eos`は以下の条件を満たさなければなりません。

- 定数`omega_a`および`omega_b`を定義している。
- 静的関数`pressure(t,v,a,b)`および`cubic_eq(a,b)`を定義している。

### `VanDerWaals`

```cpp
/// @brief Van der Waals EoS for CubicEos.
/// @tparam T Value type
template <typename T>
struct VanDerWaals {
  /// Constant for attraction parameter
  static constexpr double omega_a = 0.421875;
  /// Constant for respulsion parameter
  static constexpr double omega_b = 0.125;

  // Static functions

  /// @brief Computes pressure at given temperature and volume.
  /// @param[in] t Temperature
  /// @param[in] v Volume
  /// @param[in] a Attraction parameter
  /// @param[in] b Repulsion parameter
  /// @returns Pressure
  static T pressure(const T &t, const T &v, const T &a, const T &b) noexcept {
    return gas_constant * t / (v - b) - a / (v * v);
  }

  /// @brief Computes coefficients of cubic equation
  /// @param[in] a Reduced attraction parameter
  /// @param[in] b Reduced repulsion parameter
  /// @returns Coefficients of the cubic equation of z-factor
  static std::array<T, 3> cubic_eq(const T &a, const T &b) noexcept {
    return {-b - 1, a, -a * b};
  }
};
```

### `SoaveRedlichKwong`

```cpp
/// @brief Soave-Redlich-Kwong EoS for CubicEos.
/// @tparam T Value type
template <typename T>
class SoaveRedlichKwong {
 public:
  static constexpr double omega_a = 0.42748;
  static constexpr double omega_b = 0.08664;

  // Static functions

  static T pressure(const T &t, const T &v, const T &a, const T &b) noexcept {
    return gas_constant * t / (v - b) - a / (v * (v + b));
  }

  static std::array<T, 3> cubic_eq(const T &a, const T &b) noexcept {
    return {-1, a - b - b * b, -a * b};
  }
};
```

### `PengRobinson`

```cpp
/// @brief Peng-Robinson EoS for CubicEos.
/// @tparam T Value type
template <typename T>
class PengRobinson {
 public:
  static constexpr double omega_a = 0.45724;
  static constexpr double omega_b = 0.07780;

  // Static functions

  static T pressure(const T &t, const T &v, const T &a, const T &b) noexcept {
    return gas_constant * t / (v - b) - a / ((v - b) * (v + b) + 2 * b * v);
  }

  static std::array<T, 3> cubic_eq(const T &a, const T &b) noexcept {
    return {b - 1, a - (3 * b + 2) * b, (-a + b + b * b) * b};
  }
};
```

## ポリシー：`Corrector`

ポリシー`Corrector`は、メンバー関数`alpha(tr)`を持たなければなりません。

### `DefaultCorrector`

```cpp
template <typename T>
struct DefaultCorrector {
  T alpha(const T&) const noexcept { return 1; }
};
```

### `SoaveCorrector`

```cpp
template <typename T>
class SoaveCorrector {
  /// @brief Computes correction factor for temperature dependence of attraction
  /// parameter
  static T m(const T &omega) noexcept {
    return 0.48 + (1.574 - 0.176 * omega) * omega;
  }

  // Constructors

  /// @brief Constructs EoS
  /// @param[in] omega Acentric factor
  SoaveCorrector(const T &omega) noexcept : m_{this->m(omega)} {}

  /// @brief Computes correction factor for attraction parameter
  /// parameter
  /// @param[in] tr Reduced temperature
  /// @returns Temperature correction factor for the attraction parameter
  T alpha(const T &tr) const noexcept {
    using std::sqrt;
    const auto a = 1 + m_ * (1 - sqrt(tr));
    return a * a;
  }

 private:
  /// Correction factor for temperature dependence of attraction parameter
  T m_;
};
```

### `PengRobinsonCorrector`

```cpp
template <typename T>
class PengRobinsonCorrector {
 public:
  static T m(const T &omega) noexcept {
    return 0.3796 + omega * (1.485 - omega * (0.1644 - 0.01667 * omega));
  }

  // Constructors

  PengRobinsonCorrector(const T &omega) : m_{this->m(omega)} {}

  // Member functions

  T alpha(const T &tr) const noexcept {
    using std::sqrt;
    const auto a = 1 + m_ * (1 - sqrt(tr));
    return a * a;
  }

 private:
  T m_;
};
```

## ヘルパー関数

EOSを作成するヘルパー関数を作っておきます。

```cpp
template <typename T>
auto make_vdw_eos(const T& pc, const T& tc)
    -> CubicEos<T, VanDerWaals<T>, DefaultCorrector<T>> {
  return {pc, tc, DefaultCorrector<T>{}};
}

template <typename T>
auto make_srk_eos(const T& pc, const T& tc, const T& omega)
    -> CubicEos<T, SoaveRedlichKwong<T>, SoaveCorrector<T>> {
  return {pc, tc, SoaveCorrector<T>{omega}};
}

template <typename T>
auto make_pr_eos(const T& pc, const T& tc, const T& omega)
    -> CubicEos<T, PengRobinson<T>, PengRobinsonCorrector<T>> {
  return {pc, tc, PengRobinsonCorrector<T>{omega}};
}
```


## 使い方

この3次の状態方程式のクラスの使い方は次のようになります。

```cpp
// Critical parameters for methane
const double pc = 4e6;      // Critical pressure [Pa]
const double tc = 190.6;    // Critical temperature [K]
const double omega = 0.008; // Acentric factor

// Creates EoS object
const auto eos = make_pr_eos(pc, tc, omega);

// Computes z-factor
{
  const double p = 3e6;    // Pressure [Pa]
  const double t = 180.0;  // Temperature [K]
  // Please note that there can be multile values for z-factor.
  const auto z = eos.zfactor(p, t);
}

// Computes pressure at given temperature and volume
{
  const double t = 180.0;  // Temperature [K]
  const double v = 0.001;  // Volume [m3]
  const auto p = eos.pressure(t, v);
}
```