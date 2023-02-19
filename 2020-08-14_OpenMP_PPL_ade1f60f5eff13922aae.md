<!--
title:   OpenMPとMicrosoft PPLとParallel STLを比較してみる（その１）
tags:    C++,OpenMP,PPL,並列計算
id:      ade1f60f5eff13922aae
private: false
-->
# はじめに

今まで色々なフレームワークに乗っかってプログラミングしていて、自分で一から並列計算のコードを書いていないなと思ったので、夏休みの自由研究としてOpenMP/Microsoft PPLを使ってみました。題材は下記ウェブページにならいモンテカルロ法を使った円周率計算です。
[【c/c++】Microsoft PPL/並列STL(C++17)/OpenMPを少しだけ比較【C++17】](https://wrongwrong163377.hatenablog.com/entry/2018/07/02/003000)
とりあえずOpenMP/Microsoft PPLを使って書きましたが、そのうちMPIやParallel STLを使った場合も追加したいと思います。コードはGitHubにあげてあります。
[shohirose/openmp-examples/src/comparison.cpp](https://github.com/shohirose/openmp-examples/blob/master/src/comparison.cpp)

2020/8/15: Parallel STLの場合を追記しました。
2020/8/16: リファクタリングしました。

# コード解説

## main関数

- `PointGenerator`は`numberOfPoints`個の点を二次元領域 $\Omega = [0,1] \times [0,1]$ 内にランダムに生成します。
- `SerialCounter`/`MicrosoftPPLCounter`/`OpenMPCounter`/`ParallelSTLCounter`はそれぞれ逐次実行/Microsoft PPL/OpenMP/Parallel STLを使って単位円内に存在する点の数を数えます。
- `PiCalculator`は渡された`Counter`を使って点の数を数え円周率を計算します。
- `measureExecTime`関数は渡されたファンクターの実行時間を計測し、実行時間と計算結果を返す関数です。

```cpp
using std::chrono::duration;
using std::chrono::duration_cast;
using std::chrono::microseconds;
using std::chrono::system_clock;

std::pair<microseconds, std::vector<Point>>
measureExecTime(const PointGenerator& f) {
  const auto start = system_clock::now();
  const auto points = f();
  const auto end = system_clock::now();
  return {duration_cast<microseconds>(end - start), points};
}

template <typename Counter>
std::pair<microseconds, double>
measureExecTime(const PiCalculator &f, Counter&& counter) {
  const auto start = system_clock::now();
  const auto pi = f(counter);
  const auto end = system_clock::now();
  return {duration_cast<microseconds>(end - start), pi};
}

int main() {
  size_t numberOfPoints = 10'000'000;
  std::cout << "Number of points: " << numberOfPoints << std::endl;

  const auto [time0, points] = measureExecTime(PointGenerator(numberOfPoints));

  const PiCalculator calculator(points);
  const auto [time1, pi1] = measureExecTime(calculator, SerialCounter{});
  const auto [time2, pi2] = measureExecTime(calculator, MicrosoftPPLCounter{});
  const auto [time3, pi3] = measureExecTime(calculator, OpenMPCounter{});
  const auto [time4, pi4] = measureExecTime(calculator, ParallelSTLCounter{});

  std::cout << "Serial        : time = " << time1.count() << " usec, pi = " << pi1
            << "\nMicrosoft PPL: time = " << time2.count() << " usec, pi = " << pi2
            << "\nOpenMP       : time = " << time3.count() << " usec, pi = " << pi3
            << "\nParallel STL : time = " << time4.count() << " usec, pi = " << pi4
            << std::endl;
}
```

## `PointGenerator`

STLのrandomライブラリを使って点を生成しています。

```cpp
/// Point in two dimensions.
struct Point {
  double x;
  double y;

  Point() = default;
  Point(double t_x, double t_y) : x{t_x}, y{t_y} {}
  Point(const Point &) = default;
  Point(Point &&) = default;
  Point &operator=(const Point &) = default;
  Point &operator=(Point &&) = default;

  double rsqr() const noexcept { return x * x + y * y; }
};

// Generates random points in the region of [0,1]x[0,1]
// using OpenMP. Random numbers are generated by using STL
// random library.
class PointGenerator {
 public:
  PointGenerator(size_t numberOfPoints) : numberOfPoints_{numberOfPoints} {}

  std::vector<Point> operator()() const {
    std::vector<Point> points(numberOfPoints);
    const auto n = static_cast<std::int64_t>(numberOfPoints);

#pragma omp parallel
    {
      std::random_device generator;
      std::mt19937_64 engine(generator());
      std::uniform_real_distribution<> dist(0.0, 1.0);

#pragma omp for
      for (std::int64_t i = 0; i < n; ++i) {
        auto &point = points[i];
        point.x = dist(engine);
        point.y = dist(engine);
      }
    }
    return points;
  }

 private:
  size_t numberOfPoints_;
};
```

## `PiCalculator`

原点から半径1の円内に落ちている点の数を4倍して点の総数で割り円周率を計算します。点数は`Counter`を使って計算します。

```cpp
class PiCalculator {
 public:
  PiCalculator(const std::vector<Point> &points) : points_{points} {}

  template <typename Counter>
  double operator()(Counter &&counter) const {
    const auto numberOfPointsInCircle = counter(points_);
    const auto totalNumberOfPoints = points_.size();
    return 4.0 * numberOfPointsInCircle / totalNumberOfPoints;
  }

 private:
  const std::vector<Point> &points_;
};
```


## `SerialCounter`

`std::count_if`を使って点数を数えます。

```cpp
struct SerialCounter {
  size_t operator()(const std::vector<Point>& points) const {
    return std::count_if(begin(points), end(points),
                         [](const Point& point) { return point.rsqr() <= 1.0; });
  }
};
```

## `MicrosoftPPLCounter`

Microsoft PPLの`parallel_for_each`と`combinable`を使って円内に落ちている点の数を数えています。

```cpp
struct MicrosoftPPLCounter {
  size_t operator()(const std::vector<Point> &points) const {
    using concurrency::combinable;
    using concurrency::parallel_for_each;
    combinable<size_t> count;
    parallel_for_each(begin(points), end(points), [&count](const Point &point) {
      if (point.rsqr() <= 1.0) count.local() += 1;
    });
    return count.combine(std::plus<size_t>());
  }
};
```

## `OpenMPCounter`

点数を数えるforループをOpenMPを使って並列化しています。

```cpp
struct OpenMPCounter {
  size_t operator()(const std::vector<Point> &points) const {
    std::size_t numberOfPointsInCircle = 0;
    const auto numberOfPoints = static_cast<std::int64_t>(points.size());

#pragma omp parallel for reduction(+ : numberOfPointsInCircle)
    for (std::int64_t i = 0; i < numberOfPoints; ++i) {
      if (points[i].rsqr() <= 1.0) ++numberOfPointsInCircle;
    }

    return numberOfPointsInCircle;
  }
};
```

## `ParallelSTLCounter`

`std::count_if`に実行ポリシー`std::execution::par_unseq`を渡して計算しています。ほとんど`SerialCounter`と同じです。

```cpp
struct ParallelSTLCounter {
  size_t operator()(const std::vector<Point> &points) const {
    using std::execution::par_unseq;
    return std::count_if(
        par_unseq, begin(points), end(points),
        [](const Point &point) { return point.rsqr() <= 1.0; });
  }
};
```

# 結果

私のPC（CPU: Intel Core i7-9700K 8コア/8スレッド 3.60GHz）だと、1億個の点から円周率を計算したときの実行時間は以下の通りとなりました。

```terminal
Number of points: 100000000
Serial       : time =     116127 usec, pi = 3.14111
Microsoft PPL: time =     189072 usec, pi = 3.14111
OpenMP       : time =      51628 usec, pi = 3.14111
Parallel STL : time =      56189 usec, pi = 3.14111
```

なぜかMicrosoft PPLがSerialよりも遅い。どこか何か間違っているのかな…。どなたか理由がわかるかたがいらっしゃったらコメント頂けると幸いです。