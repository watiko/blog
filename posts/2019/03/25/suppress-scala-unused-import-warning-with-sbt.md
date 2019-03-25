---
title: "sbt console の際に unused import で怒られないようにする"
date: 2019-03-25T23:13:51+09:00
type: post
---

[sbtで "-Ywarn-unused" や "-Ywarn-unused-import" を設定する際のベストプラクティス - xuwei-k's blog](https://xuwei-k.hatenablog.com/entry/20150417/1429242775)

この方法は古くなっており動かなかったので色々調べた結果。

<!--more-->

まず、 scalac の最新のオプションでどうなっているかを確認する。この際にバージョンごとにオプションが違ったりすることがあるので注意すること。

```bash
$ scalac -version
Scala compiler version 2.12.8 -- Copyright 2002-2018, LAMP/EPFL and Lightbend, Inc.
$ scalac -Y
...snip...
  -Ywarn-unused:<_,warning,-warning>       Enable or disable specific `unused' warnings: `_' for all, `-Ywarn-unused:help' to list choices.
...snip...
$ scalac -Ywarn-unused:help
$ Enable or disable specific `unused' warnings
  imports    Warn if an import selector is not referenced.
  patvars    Warn if a variable bound in a pattern is unused.
  privates   Warn if a private member is unused.
  locals     Warn if a local definition is unused.
  explicits  Warn if an explicit parameter is unused.
  implicits  Warn if an implicit parameter is unused.
  params     Enable -Ywarn-unused:explicits,implicits.
  linted     -Xlint:unused.
Default: All choices are enabled by default.
```

こんな感じにコンパイラに確認するのが確実です。

どうやら今使っているバージョンでは `-Ywarn-unused:-imports,_` のような感じで指定すると import が未使用でも怒られなくなりそうです。  
実際のアプリケーションでは未使用インポートはないほうが良いので、 以下のように build.sbt に記述をしました。(ベースの設定は [scala-seed.g8](https://github.com/scala/scala-seed.g8) で設定したものです)

```scala
scalacOptions ++= Seq(
  "-encoding",
  "UTF-8",
  "-deprecation",          // warn about use of deprecated APIs
  "-unchecked",            // warn about unchecked type parameters
  "-feature",              // warn about misused language features
  "-language:higherKinds", // allow higher kinded types without `import scala.language.higherKinds`
  "-Xlint",                // enable handy linter warnings
  "-Xfatal-warnings",      // turn compiler warnings into errors
  "-Ypartial-unification"  // allow the compiler to unify type constructors of different arities
)

// sbt console した時に import ができるようにしている。
Seq(Compile, Test).flatMap(scalacOptions in (_, console) += "-Ywarn-unused:-imports,_")
```

このような設定が行われているプロジェクトで `sbt console` をして import を行った際に警告が発生しなければうまく設定ができています。

余談ですが、ペーストモードで塊としてスニペットを実行する分には問題がないので設定を変えるほどではない場合はそちらでしのぐというのもアリかもしれません。
