---
title: "C++20 の Concept は型クラス？"
date: 2019-02-25T23:25:00+09:00
type: post
url: /2019/02/26/cpp_concept
---

C++20 から導入される予定の機能に Concept というものがあります。
どういうものかというと型の継承関係に依存せずにある型が満たすべき性質を記述できるものです。

この辺りの機能を使うと型クラスのようなことができそうなので試してみました。

型クラスという言葉を初めて聞いた人向けに以下の記事をお勧めします。

- Cats: https://typelevel.org/cats/typeclasses.html
- ドワンゴ: https://dwango.github.io/scala_text/introduction-to-typeclass.html

<!--more-->

Concept の詳細な説明は下記のサイトに譲るとして、ここでは簡単な紹介といきなり具体的なコードに移ろうと思います。
(短いコード片などは以下のサイトから引用しています)

- https://en.cppreference.com/w/cpp/language/constraints

また、コンパイルできる環境を用意するのは大変なので [Wandbox](https://wandbox.org/) を使用します。

```cpp
template<typename T>
concept bool Addable = requires (T x) { x + x; };
 
template<typename T> requires Addable<T>
T add(T a, T b) { return a + b; }
```

このようにするとある型 `T` の値に `+` という `T` に閉じた二項演算子を要求する `Addable` というコンセプト、
それから、`Addable` を用いて型に制約をつけている関数 `add` が定義できます。

```cpp
int main()
{
    std::cout << add("", "") << std::endl;
}
```

このような記述をした上でコンパイルすると以下のようなエラーが出力されます。
[試してみたい方はこちら](https://wandbox.org/permlink/nsQsvAG3o637gcuI)

```
prog.cc: In function 'int main()':
prog.cc:12:28: error: cannot call function 'T add(T, T) [with T = const char*]'
   12 |     std::cout << add("", "") << std::endl;
      |                            ^
prog.cc:8:3: note:   constraints not satisfied
    8 | T add(T a, T b) { return a + b; }
      |   ^~~
prog.cc:5:14: note: within 'template<class T> concept const bool Addable<T> [with T = const char*]'
    5 | concept bool Addable = requires (T x) { x + x; };
      |              ^~~~~~~
prog.cc:5:14: note:     with 'const char* x'
prog.cc:5:14: note: the required expression '(x + x)' would be ill-formed
```

さて、なんとなく Concept の気持ちがわかってきたところで本題に入ります。

早速ですが、普段 C++ 書かないなりにひねり出したのが以下のコードです。

{{< gist watiko 88f9c1f5a64cc6805e07774d5e91305a "concept.cpp" >}}
https://wandbox.org/permlink/nsQsvAG3o637gcuI

- OrdTraits: テンプレートの特殊化(traits パターンと呼ばれるらしい)
  - ここでは `int` に対して実装を用意しています
- max の定義に使っている `requires Ord<T>` はなくてもコンパイルができます。
  - ただし、その場合に想定外の型に使われた場合のエラーがわかりづらい

つまり traits パターンを用いることでもともと型クラスに近いことができていたが、
Concept が導入されることでより安全で分かりやすいコードになったということのようです。

残念なポイントとして、 `OrdTraits` と `Ord` の間で名前が衝突してしまうので二つの名前を使い分ける必要があるところが挙げられます。
また、構文的にも非常に分かりずらいので(特に traits 周り)実用したいかと言われるとなんとも言えないところですね。

## おまけ

Scala で上記のサンプルと同じことをかくとこんな感じ。(Wandbox で動かせる)

{{< gist watiko 88f9c1f5a64cc6805e07774d5e91305a "implicit.scala" >}}
