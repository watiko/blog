---
title: "Rust の cargo プロジェクトで複数のファイルを実行可能にする方法"
date: 2019-03-23T20:58:56+09:00
type: post
---

先週あたりから Rust (再)入門したのでその際の覚書です。

プログラミング言語に入門している時っていろんな構文や機能を試すためにファイルをたくさん作成してそれぞれ実行したりしたいですよね？

<!--more-->

残念なことに標準のツールでは Rust のファイルをスクリプト的に実行させることはできないようです。また、外部のライブラリに依存したい場合は結局 `cargo` を使用する必要があるので `cargo` から実行するということに焦点を当てます。

いきなり結論ですが、プロジェクトの `src/bin/*.rs` にファイルを作成すると `cargo` の規約によりそのファイルは実行可能だという風に認識されます。細かいことや `config.toml` の `[[bin]]` との兼ね合いはリンク先を読んで欲しいのですが、重要な部分だけ抜粋します。

https://doc.rust-lang.org/cargo/reference/manifest.html#the-project-layout

> Cargo will also treat any files located in src/bin/*.rs as executables.

例えば以下のようなことが可能です。

```bash
$ cargo new bin-proj
$ cd bin-proj && tree
bin-proj
├── Cargo.toml
└── src
    └── main.rs
1 directory, 2 files
$ mkdir src/bin
$ vim src/bin/hello.rs
$ cat src/bin/hello.rs
fn main() {
    println!("hello bin dir!");
}
$ cargo run --bin hello
   Compiling hello-bin v0.1.0 (/private/tmp/rust/aaa)
    Finished dev [unoptimized + debuginfo] target(s) in 0.38s
     Running `target/debug/hello`
hello bin dir!
```

`cargo` に実行可能として認識されていると、 IntelliJ でもきちんと認識されるので非常に便利になります。
