# ムーブセマンティクス

## コピーとムーブ

```cpp
std::vector<int> x(1000000);
std::vector<int> y = x; // ディープコピー

// 以降 x は利用しない
```

この例の場合、`y` を生成するために `x` と同程度のメモリ領域を確保する必要があります。
これはコストの面ではかなりの無駄があります。

```cpp
std::vector<int>* x = new std::vector<int>(1000000);
std::vector<int>* y = x; // シャローコピー
x = nullptr;

// 以降 x は利用しない
```

この例では `x` を `y` にシャローコピーした後に、 `x` に `nullptr` を代入しています。

これがムーブの根底にある考え方で、
ポインタの付け替えだけで、あたかも `x` が `y` に移動しているような挙動を実現出来ているため、
ディープコピーのときよりもコストの面で有利です。

しかし、ポインタの操作を伴う実装になるため、プログラマが細心の注意を払って実装しないと、
ムーブの仕組みが実現できなくなるというリスクも存在します。

そこで、 C++11からはムーブの考え方をテクニックとしてでは無く、言語仕様として実現する仕組みが取り入れられました。

## 右辺値と左辺値

- 左辺値: 基本的にそのスコープの間生き続ける名前付きのオブジェクト
- 右辺値: リテラルや関数が返す一時オブジェクトのようなその瞬間に破棄されて不要になるもの

```cpp
int x = 300; // x は左辺値。300 のようなリテラルは右辺値。
int y = function(); // y は左辺値。関数 function が返す一時オブジェクトは右辺値。
int z = x; // z と x は左辺値。
```

右辺値は左辺値へ格納したら必要無くなるので、右辺値にはムーブの考え方が適用できることがわかります。
C++11 以降、コンパイラはこの考え方を認識し、ムーブを実現してくれるようになりました。

## std::move

次の例では、`x` はコンパイラからしてみると左辺値なので、`y` への代入時にディープコピーされます。

```cpp
std::vector<int> x(1000000);
std::vector<int> y = x; // y と x は左辺値

// 以降 x は利用しない
```

そこで、左辺値を右辺値にキャストする `std::move` を使います。
`x` を右辺値にキャストすることで、ディープコピーではなくムーブができるようになります。

```cpp
std::vector<int> x(1000000);
std::vector<int> y = std::move(x); // std::move で x を右辺値にキャスト

// 以降、xは利用しない
```

## 右辺値参照

右辺値を参照するには `&&` を付けます。

```cpp
int&& x = 100;
```

左辺値の参照と右辺値の参照は別の型として扱われるため、
オーバーロードの仕組みを利用することで、引数が左辺値参照の時と右辺値参照の時で処理を分けることが可能です。

```cpp
#include <iostream>
#include <string>

void Print(std::string&) {
    std::cout << "左辺値参照" << std::endl;
}

void Print(std::string&&) {
    std::cout << "右辺値参照" << std::endl;
}

int main() {
    std::string str = "hoge";

    Print(str);
    Print(std::move(str));

    return 0;
}
```

```bash
# 実行結果
左辺値参照
右辺値参照
```