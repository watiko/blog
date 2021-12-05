---
title: "zshの設定を見直し"
date: 2021-12-05T18:37:48+09:00
type: post
url: /2021/12/05/renew-zshrc
---

# zshの設定を見直し

調べたことのメモを兼ねて書き残しておく。

## あらまし

人に言われてzshの起動が遅いことに気がついたのがきっかけで設定の見直しを行なった。当時の設定は残していないがおおよそ2000msほどかかっており、ターミナルの分割やIDEの中でターミナルを開く毎に待ち時間が発生していた。言われると気になり始めたのでその場で簡易に対処をした(nvmからasdfに変えて1000ms程度になった)。また、年末の大掃除ということでzinitを導入してオーバーホールも行った(最終的に90ms程度になった)。

## 以前の運用

基本的にはどの環境(macOS, Linux, WSL, etc...)でもpreztoを入れて、その場で必要なツール(nvm, rbenv, etc...)の設定をそれぞれ追加していくという方針をとっていた。これはこれで気を使わなくて良いので楽ではあった。一応git周りはどの環境でもいじることが多かったので `.gitconfig` は手動で共有したりしていた。

## 計測いろいろ

まずは `.zshrc` のどこが遅いのかを調べて、遅い部分から潰していく。
以下で紹介する計測手法とコメントアウトなどを駆使して遅い部分を探す。

### 前提知識

計測に関係するzshのオプション。

- `-i` or `--interactive`: `zprofile`, `zlogin` を読み込む
- `-l` or `--login`: `zshrc` を読み込む
- `-c`: 渡したコマンド列を起動した環境で実行する

`zshenv` についてはどの組み合わせでも読み込む。

#### 参考

読み込み順などについては `man zsh` の `STARTUP/SHUTDOWN FILES` より。
>Commands are then read from $ZDOTDIR/.zshenv.  If the shell is a login shell,
>commands are read from /etc/zprofile and then $ZDOTDIR/.zprofile.  Then, if the
>shell is interactive, commands are read from /etc/zshrc and then $ZDOTDIR/.zshrc.
>Finally, if the shell is a login shell, /etc/zlogin and $ZDOTDIR/.zlogin are read.

### zshのプロファイリング機能を使う方法

関数毎の呼び出し回数やかかっている時間などを確認する `zprof` を使用する。(詳細は `man zshmodules`)

```shell
# zshrcの冒頭に書く
zmodload zsh/zprof

...snip...

# zshrcの最後に書く
if type zprof >/dev/null 2>&1; then
  zprof | less
fi
```

### 特定箇所の時間を計測する方法

```shell
# 計測開始地点に書く
zmodload zsh/datetime
function get_time_ms() {
  strftime '%s%.'
}
function show_time() {
  local start=$1
  local end=$2
  echo $((end - start))
}
start_time=$(get_time_ms)

...snip...

# 計測終了地点に書く
end_time=$(get_time_ms)
show_time $start_time $end_time
```

この表示を利用して平均値を取得する方法。

```shell
repeat 10 { zsh -i -c exit } \
  | awk '{ total += $1 } END { print total/NR }'
```

### 全体的に測る

`~/.zshrc` に限らずそのほかのファイルも含めた起動時間を測る方法。

[TIMEFMT](https://zsh.sourceforge.io/Doc/Release/Parameters.html)を設定することでzsh組み込みのtimeの出力を制御できる。`%mE` を指定してミリ秒の値を得ている。(1000msとかをawkがいい感じに数字として解釈していそう)

```shell
( export TIMEFMT='%uE'; repeat 10 { time zsh -i -l -c exit } 2>&1 ) \
  | awk '{ total += $1 } END { print total/NR }'
```

### 参考: hyperfine

コマンドの実行速度の計測に便利なツールとして [hyperfine](https://github.com/sharkdp/hyperfine) があるのでついでに紹介。

```shell=
hyperfine 'zsh -i -l -c exit' 'zsh -i -c exit' 'zsh --no-rcs -c exit'
Benchmark 1: zsh -i -l -c exit
  Time (mean ± σ):      86.9 ms ±   3.9 ms    [User: 54.0 ms, System: 27.3 ms]
  Range (min … max):    82.2 ms …  96.8 ms    34 runs

Benchmark 2: zsh -i -c exit
  Time (mean ± σ):      80.6 ms ±   3.2 ms    [User: 52.6 ms, System: 23.2 ms]
  Range (min … max):    75.5 ms …  91.9 ms    33 runs

Benchmark 3: zsh --no-rcs -c exit
  Time (mean ± σ):       6.3 ms ±   0.7 ms    [User: 2.5 ms, System: 1.9 ms]
  Range (min … max):     4.2 ms …   8.5 ms    329 runs

  Warning: Command took less than 5 ms to complete. Results might be inaccurate.

Summary
  'zsh --no-rcs -c exit' ran
   12.77 ± 1.42 times faster than 'zsh -i -c exit'
   13.78 ± 1.56 times faster than 'zsh -i -l -c exit'
```

## 最適化

zinitを用いて補完設定などを積極的に遅延実行させることで起動にかかる時間を短縮していく。単純に実行速度が遅いコマンドなどがあればそれを除去するのも効果がある。

### zinit

この記事ではzinitについての基本などはあまり解説しないので、公式のドキュメントや他の人が書いた記事を参考にすること。(この記事の中ではzinitのmodifier指定の際に `modifier=value` という構文を使っている)

- 公式
  - [zdharma-continuum/zinit](https://github.com/zdharma-continuum/zinit)
  - [Zinit Wiki](https://zdharma-continuum.github.io/zinit/wiki/)
- おすすめの記事
  - [zinit をしっかりと理解する](https://zenn.dev/xeres/articles/2021-05-05-understanding-zinit-syntax)

### 補完設定

zshの補完周りの詳しい話は `man zshcompsys` を見ること。zinit固有の話はリポジトリの[Completions](https://github.com/zdharma-continuum/zinit#completions-2)にまとまっている。

zinitの補完周りは `compdef` を差し替えて `compinit` 後に改めて `zinit cdreplay` (`zicompinit`) で適用するような形となる。この時に自分の制御外で `compinit` を呼び出すようなスクリプトが存在すると速度に影響が出るため気をつける必要がある。(`zprof` を使って確認できる)

また、補完周りのスクリプトの実行時間は長いことが多いため、遅延実行するとよい。zinitではwait modifierを設定することでプロンプト表示後に実行を遅延できる。詳しくは[Wiki](https://zdharma-continuum.github.io/zinit/wiki/Example-wait-conditions/)を見てほしいが、`0a`, `0b`, `0c` として表示直後に実行する中でもある程度の実行順制御ができるので、補完系の順序制御はこれを活用する。(順序を制御せずに遅延実行だけした場合にどうなるかというのはあまりわかっていないが、プロンプト表示後に補完が有効になるまでの時間が伸びたりする懸念はある)

私は以下のような形で活用している。

| wait | 用途 |
|------|-----|
| `0a` | `compdef` を呼ぶ |
| `0b` | `zicompinit` を呼ぶ |
| `0`  | そのほか遅延実行したいもの |

```shell
zinit as="command" wait="0a" lucid light-mode for \
  pick="asdf.sh" src="completions/_asdf" @asdf-vm/asdf

# compinit呼び出し後に実行されてほしいスクリプト
# (今はzinit管理になっていないスクリプトが押し込められている)
function after_completion_setup() {
  autoload -Uz +X bashcompinit && bashcompinit

  [[ -f /usr/local/bin/aws_completer ]] && complete -C /usr/local/bin/aws_completer aws
  [[ -f "$GCLOUD_HOME/completion.zsh.inc" ]] && "$GCLOUD_HOME/completion.zsh.inc"
}

zinit wait="0b" lucid light-mode for \
  atload="zicompinit; zicdreplay; after_completion_setup" \
  zsh-users/zsh-syntax-highlighting
```

## 結果

執筆時点の内容は以下のようになっておりおおよそ90msを下回る速度で起動するようになった。 `PZTM::completion` を呼んでいる部分や `sdkman`, `gcloud` 周りなどあまり考えず適当に処理している部分もあるが、手元では素早く起動する上で、問題も発生していないので一旦記録を残そうということでこの記事を書いた。zinit周りの細かいテクニックは各自で理解して欲しい。

- [watiko/dotfiles/zshrc]( https://github.com/watiko/dotfiles/blob/1753d189fc8ab5848d7a3fd103e0d3dc80eb9802/zshrc)

```shell
$ hyperfine 'zsh -i -l -c exit'
Benchmark 1: zsh -i -l -c exit
  Time (mean ± σ):      84.9 ms ±   3.7 ms    [User: 52.3 ms, System: 26.8 ms]
  Range (min … max):    82.1 ms …  99.9 ms    34 runs
```

## 参考

- [Faster and enjoyable ZSH (maybe)](https://htr3n.github.io/2018/07/faster-zsh/)
