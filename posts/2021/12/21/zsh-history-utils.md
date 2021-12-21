---
title: "zshのヒストリーファイルを変換するツールを作った"
date: 2021-12-21T00:00:00+09:00
type: post
url: /2021/12/21/zsh-history-utils
---

## 要約

[watiko/zsh-history-utils](https://github.com/watiko/zsh-history-utils) を作ったよ。

## zsh のヒストリーファイルとは

zsh を使っているときに `Ctrl-R` などを押して過去に実行したコマンドを再利用することがありますが、あの履歴はファイルに保存されています。
個々人の設定によって違いますが現在どのファイルを使用しているかというのは `$HISTFILE` を確認するとわかります。

フォーマット自体はデフォルトのものと [EXTENDED_HISTORY](https://zsh.sourceforge.io/Doc/Release/Options.html#index-EXTENDEDHISTORY) によって設定される拡張されたものとがあります。ここでは拡張されたものの方だけ取り扱うことにします。

> Save each command’s beginning timestamp (in seconds since the epoch) and the duration (in seconds) to the history file. The format of this prefixed data is:
>
> ‘: \<beginning time\>:\<elapsed seconds\>;\<command\>’.

非常にシンプルなフォーマットのように見えますが、実際は違います。詳細は後述しますがこの問題に対処していない既存のツールなども見受けられました。

### そもそもなぜヒストリーファイルをいじる必要があったか

[前回の記事](/2021/12/05/renew-zshrc/)の作業中に HISTFILE の向き先が変わってしまっており、ヒストリーファイルが複数に分割されてしまったのが発端です。
コマンドの実行履歴の参照は日常的に行なっているため、これが分散してしまい不便になったのでなんとかしてこれをマージしようと考えました。

単に一覧を(実行時間は微妙ですが)出力するだけならば以下のように組み込みの `fc` コマンドで可能です。ただし、(私が知る限り)これを元のフォーマットに戻すことはできません。

```zsh
$ fc -l -n -t '%Y-%m-%dT%H:%M:%S.%fZ' -D 0 | tail -n 2
2021-12-20T23:23:13.20Z  0:00  echo aaaa
2021-12-20T23:23:26.20Z  0:04  sleep 4
```

また、フォーマット的には `cat` で複数のファイルを結合するだけでも正しいものを得ることができます。ただし、履歴の順序が入れ替わったりするため履歴の参照を行なった際の順序が問題となり得ます。私の場合は以前使っていたヒストリーファイルも残っており、そのファイルに新たに書き込むよう設定してしまったため、単純に `cat` でやるのはどうもなぁという気分になっていました。(複数台のマシンの履歴をマージする際も似たような問題に出くわすと思われます)

## 使用例

最終的に `zsh-history-utils` というコマンドラインのツールを作りました。`decode`, `encode`, `merge` という 3 つの操作を実装しており、JSON としてヒストリーファイルを書き出す機能を作ったのが思ったよりも便利に使えたのでその辺の話もします。

### マージ操作

元々やりたかったヒストリーファイルのマージ処理は以下のように行うことができます。

```zsh
# >| は上書きもするリダイレクトです
$ zsh-history-utils merge ~/.zhistory ~/.zsh_history >| /tmp/merged.history
# (push) 一時的に指定したファイルをヒストリの書き出し先にできる
$ fc -p /tmp/merged.history
# (pop) 問題がなさそうであれば、元々の向き先に戻す
$ fc -P
$ cp /tmp/merged.history $HISTFILE
```

### フィルタ処理

`decode` はヒストリーファイルを JSON にする処理、 `encode` はその逆を実装しています。つまり `decode` して `encode` すると元に戻るわけですが、間に JSON を噛ませているので `jq` などを用いて簡単にフィルタを行うことが可能です。
例えば、コピペミスで貼り付けてしまった長大なログや、誤ってヒストリに残してしまっている情報、一時的なトークン情報(export)などを削除した状態のヒストリーファイルを保つことが簡単になります。コマンドを参照する際に `fzf` などを噛ませていると長い履歴はそれだけで引っかかりやすいので不要な履歴を削除すると結構快適になります。

以下は長すぎる履歴を除去するコマンド例です。(150 という数字は後述するヒストグラムをもとに私の手元でいい感じだと思われた数値です)

```zsh
zsh-history-utils.rs decode $HISTFILE \
  | jq -c 'select(.command | length < 150)' \
  | zsh-history-utils.rs encode -
  >| /tmp/merged.history
```

#### 他のアイディア

- コピペで実行したコマンドの意図せぬ改行が履歴に残っているものを修正
- 重複したコマンド実行の削除

### ヒストグラムを眺める

コマンドラインでグラフを書くツールはいろいろありますがここでは [YouPlot](https://github.com/red-data-tools/YouPlot) を使ってみます。

```zsh
$ zsh-history-utils decode $HISTFILE | jq '.command | length' | uplot hist
                  ┌                                        ┐
   [  0.0,  10.0) ┤▇▇▇▇▇▇▇▇▇▇▇ 1840
   [ 10.0,  20.0) ┤▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 5459
   [ 20.0,  30.0) ┤▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 4500
   [ 30.0,  40.0) ┤▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 3117
   [ 40.0,  50.0) ┤▇▇▇▇▇▇▇▇▇▇▇▇ 1916
   [ 50.0,  60.0) ┤▇▇▇▇▇▇▇▇ 1238
   [ 60.0,  70.0) ┤▇▇▇▇▇ 747
   [ 70.0,  80.0) ┤▇▇▇▇ 593
   [ 80.0,  90.0) ┤▇▇▇ 441
   [ 90.0, 100.0) ┤▇▇ 338
   [100.0, 110.0) ┤▇▇ 278
   [110.0, 120.0) ┤▇ 234
   [120.0, 130.0) ┤▇ 174
   [130.0, 140.0) ┤▇ 110
   [140.0, 150.0) ┤▇ 134
   [150.0, 160.0) ┤ 7
                  └                                        ┘
                                  Frequency
```

## フォーマットの話

この話を複雑にする要因は 2 つあります。1 つ目はエンコーディング、2 つ目は改行です。それぞれ順に見ていきましょう。

### エンコーディング

[hist.c のコメント](https://github.com/zsh-users/zsh/blob/6b763233b2d7db08ed4c16400356d7deb292fe50/Src/hist.c#L2712-L2722)にもあるように意味がわからないですがヒストリーファイルの中身は `metafy` という処理がされた後のバイト列が格納されています。`less $HISTFILE` できるという声もあるでしょうが、実際はすべての文字に特殊なエンコーディングが施されるわけではないため ascii の範囲内であればプレインテキスト(UTF-8)と言っても差し支えはありません。しかし日本語の文字列などは `metafy` によって UTF-8 として解釈できないバイト列になるため単純にファイルを文字列として処理するとエラーが出て処理できない場合があります。また、先ほどのフィルタ処理的にも都合が悪いでしょう。

では `metafy` とはいったいなんなのでしょうか。きちんと調べていないですが、何かしらの理由で行われている zsh の内部的な文字列表現のようです。最初にヒストリーファイルが実装される際に `unmetafy` されていればよかったのですが、そういう世界線ではないようです。具体的には以下のような処理が行われます。

```c
// 以下より抜粋して改変(エラー処理や、ヒープ周りの処理の省略)。コメントは筆者のもの
// https://github.com/zsh-users/zsh/blob/6b763233b2d7db08ed4c16400356d7deb292fe50/Src/utils.c#L4821

char* metafy(char *buf, int len, int heap)
{
    // 処理対象がいくつあるか
    int meta = 0;
    for (e = buf; e < buf + len;)
        // imetaは渡された文字が処理対象かを判定するマクロ
        // `short int typtab[256]` というものがありこれを元に判定をする
        if (imeta(*e++))
          meta++;

    // バッファを増やす
    buf = zrealloc(buf, len + meta + 1);

    while (meta) {
        if (imeta(*--t = *--p)) {
            *t-- ^= 32;
            *t = Meta;
            meta--;
        }
    }
    *e = '\0';
    return buf;
}
```

現代的なプログラミング手法に慣れた人にとっては困ってしまうようなスタイルのプログラムですが、これを Rust で書くと次のようになります。

```rust
// https://github.com/watiko/zsh-history-utils/blob/af9eceefa673af0ab2a0f91bf61a4d73bffa314d/src/zsh/core.rs#L61-L75

pub fn metafy(str: &[u8]) -> Vec<u8> {
    let mut buf = vec![];

    for &c in str {
        if !is_meta_char(c) {
            buf.push(c);
            continue;
        }

        buf.push(META);
        buf.push(c ^ META_MASK);
    }

    buf
}
```

はい。実際はそこまで複雑な処理をしているわけではないですね。 `unmetafy` もこれの逆操作なので難しくはありません。ただ、この処理相当を組み込んでいるヒストリを管理するツールがどれほどあるかは疑問です。(一見ただのプレインテキストに見えるのでしょうがないですね)

### 改行

改行の取り扱いはむずかしいです。実は曖昧になるケースがあるということで 2021/11/29 に出力側の修正が入っています。以下は関連するコミットやメーリングリストのアーカイブ。

- [49601: don't create ambiguous history file entries for lines ending with a backslash](https://github.com/zsh-users/zsh/commit/78958c08bfdb37d2eafaf14a33b93229b1fa9e31)
  - [Re: [BUG] Trailing backslash in history file confuses entries](https://www.zsh.org/mla/workers/2021/msg01823.html)
- [32882 (cf. Augie Fackler 32879): correct reload of backslash-continuation lines from history, fix bad history write of events ending with backslashes](https://github.com/zsh-users/zsh/commit/b63ff19dbf8f220f3ae8ab2ab41058f3149bde1f)
  - [Re: [PATCH] Fix loading of multi-line history entires from disk](https://www.zsh.org/mla/workers/2014/msg00670.html)

実際は以下のようなロジックでバイト列を生成します。(上記の修正が適用されているバージョンであれば以下の通りですが、古いバージョンは違います。ただし、読み込みのロジックに変更がないので実用上問題になることはないはずです)

```rust
// https://github.com/zsh-users/zsh/blob/6b763233b2d7db08ed4c16400356d7deb292fe50/Src/hist.c#L3021-L3040
// https://github.com/watiko/zsh-history-utils/blob/af9eceefa673af0ab2a0f91bf61a4d73bffa314d/src/zsh/history.rs#L97-L120

pub struct HistoryEntry {
    pub start_time: u64,
    pub finish_time: u64,
    pub command: String,
}

impl HistoryEntry {
    pub fn to_bytes(&self) -> Vec<u8> {
        let duration = self.finish_time - self.start_time;

        let mut read_buf = format!(": {}:{};", self.start_time, duration).into_bytes();
        read_buf.extend_from_slice(&metafy(self.command.as_bytes()));

        let mut buf = vec![];
        let mut end_backslashed = false;
        for c in read_buf {
            // 複数行ある場合で、最後の行がバックスラッシュで終わっているかどうか
            // ただし、バックスラッシュの後ろに空白が続いても良い
            end_backslashed = c == b'\\' || (end_backslashed && c == b' ');

            if c == b'\n' {
                // 履歴の区切りの改行と区別がつくようにバックスラッシュをつける
                // 言い換えると、 b'\\', b'\n' という並びは一つの履歴が改行されていることを意味する
                buf.push(b'\\');
            }
            buf.push(c);
        }

        if end_backslashed {
            // これがないと履歴の中の改行と見分けがつかない
            // エスケープとしての空白なのでパース時には除去する必要がある
            buf.push(b' ');
        }
        // 履歴の区切り
        buf.push(b'\n');

        buf
    }
}
```

流石にこの実装はアドホック感がありますが仕方ありません。

## 何で実装するか

### Deno

最初は Deno で作り始めました。ある程度 Node の資産も活かせる上に、formatter(`deno fmt`)や lint(`deno lint`)の導入や設定などで悩むことがないのは非常に快適です。

- [2021 年の Deno の変更点やできごとのまとめ](https://zenn.dev/uki00a/articles/whats-new-for-deno-in-2021)

#### 不具合

一方、利用者がそこまで多くないためか割と不具合を踏むことが多いです。これについては他の Deno 使用者にも聞いてみたい気持ちがあります。前回程々のサイズのスクリプトを書いたときもランタイムの挙動が怪しかったりと問題を踏む頻度が高いので。

- [namaspace まわりの不具合](https://github.com/denoland/deno/issues/12731)
- [v8 まわりの不具合](https://github.com/denoland/deno/issues/12885)

#### 設計

割と zsh のコードの構造をそのまま持ってきたため、履歴単位でシリアライズ・デシリアライズできないコードになってしまいました。
元が行(LF)ごとに前処理(`readhistline`)をしてから別の処理(`readhistfile`)をするような感じのコードになっているためですね。

#### テスト

テストを書く際にサブテストを書けるようになっているのは非常に便利でした。現状は unstable ですが次のリリースで stable になる見込みとのこと。

- [Deno の test でネストしたテストを書く方法](https://shinshin86.hateblo.jp/entry/2021/12/19/080105)

#### 標準ライブラリ

上記のテスト周りを見ていても思いましたが、Deno は全体的に Go を参考にしているようでなかなか筋が良いなと思いました。[標準ライブラリ](https://deno.land/std)などはまさに Go のライブラリのポートみたいになっているものも多く、Node の標準ライブラリよりも使いやすいと感じるものが多いです。(Node に関して言えばだいぶ時代も古いというのもありますね。当時は Promise とかもなかったわけですからね。)

#### タイプチェックにかかる時間の増大

大きいライブラリに対して依存をすると実行前のタイプチェックにかかる時間が無視できないほど大きくなることがありました。具体的には 10 秒ぐらいまで時間が伸びて流石に体験が悪かったです。私は知らなかったのですが、ちょうど `--no-check=remote` というオプションが使えるようになっていたので次からはこれを使うと良さそうです。私は `--no-check` ですべて切っていました。

#### 依存関係の管理

以前書いた時には依存ライブラリのバージョン管理が面倒そうだなという印象があったのですが、 [import-maps](https://github.com/WICG/import-maps) が使えるようになっていたのでだいぶ楽でした。

- [依存モジュールを一元管理しよう](https://zenn.dev/uki00a/books/effective-deno/viewer/manage-deps-in-a-centralized-way)
- [今の依存関係管理方法は？import map？import maps？って何？](https://zenn.dev/lion/articles/6a0501c11eb7d7)

#### 各種ツール

npm 的なレイヤを提供するツールもいくつかあるようで今回は `trex` を使ってみました。

- [crewdevio/Trex](https://github.com/crewdevio/Trex)
- [denosaurs/denon](https://github.com/denosaurs/denon)
- [Velocidex/velociraptor](https://github.com/Velocidex/velociraptor)

#### まとめ

いろいろ書いてきましたが、Deno で書くのはなかなか快適でした。ただ、微妙に遅いなと感じたので早そうな言語で書き直すことにしました。(今回は Rust)

### Rust

Deno で実装した後だったので全体的な設計とかはそこまで悩むことはなかったです。特にテストケースは持ってくるだけだったので楽でした。また、シリアライズ・デシリアライズが対応していない問題についても対処できたのはよかったポイントです。その分コードの複雑性が相当上がってしまったのはなんとも言えないですが。

具体的には `nom` を使ってバイト列をパースする際、条件に応じて[パーサーの実装を切り替えているあたり](https://github.com/watiko/zsh-history-utils/blob/master/src/zsh/history.rs#L43-L49)が微妙な感じになってしまった。(nomに慣れている人だともっとまともな実装にできるのだろうか)

### 比較

#### 速度

最終的には 50 倍程度 Rust の方が早かったです。

```zsh
$ hyperfine \
  'zsh-history-utils.deno merge .zhistory .zsh_history > /dev/null' \
  'zsh-history-utils.rs merge .zhistory .zsh_history > /dev/null'
Benchmark 1: zsh-history-utils.deno merge .zhistory .zsh_history > /dev/null
  Time (mean ± σ):      3.480 s ±  0.106 s    [User: 3.271 s, System: 0.350 s]
  Range (min … max):    3.311 s …  3.677 s    10 runs

Benchmark 2: zsh-history-utils.rs merge .zhistory .zsh_history > /dev/null
  Time (mean ± σ):      72.8 ms ±  19.1 ms    [User: 59.8 ms, System: 9.2 ms]
  Range (min … max):    61.2 ms … 162.5 ms    34 runs

  Warning: Statistical outliers were detected. Consider re-running this benchmark on a quiet PC without any interferences from other programs. It might help to use the '--warmup' or '--prepare' options.

Summary
  'zsh-history-utils.rs merge .zhistory .zsh_history > /dev/null' ran
   47.80 ± 12.63 times faster than 'zsh-history-utils.deno merge .zhistory .zsh_history > /dev/null'
```

#### Rust のテスト周りが貧弱な話

標準のテスト出力が見づらい。辛すぎて [pretty_assertions](https://github.com/colin-kiegel/rust-pretty-assertions) を入れました。
また、テーブルドリブンなテストは本当にやりづらいと感じました。一応それをサポートする create はいくらかあります。
しかし根本的にテストケースの定義の仕方が、関数に `#[test]` を付与する方式なため、ネストしたテストケースが作れません。そのため、マクロでなんとかするしかなく厳しいです。
今回はテストケースの使い回しもしたかったため、 [rstest](https://github.com/la10736/rstest) を使いました。ドキュメントに乗っているような、簡単なケースであればまあ耐えられますが、今回のように[そこそこ複雑なもの](https://github.com/watiko/zsh-history-utils/blob/af9eceefa673af0ab2a0f91bf61a4d73bffa314d/src/zsh/history.rs#L187-L251)だとかなり辛いです。

どう辛いかというと、フォーマットが効かない、コード補完が効かないということに尽きます。どうにかなりませんかね。ならないか……

後は `rstest_reuse` を使う場合 `cargo clippy --tests` がまともに動かない問題があり、これは未解決です。

## 感想

普段やらないバッファとかメモリアロケーションとかを意識するようなプログラミングをしたので新鮮な気分でコードを書いていました。
後は古い時代に書かれた C 言語のコード読むのしんどいということぐらいですかね。
