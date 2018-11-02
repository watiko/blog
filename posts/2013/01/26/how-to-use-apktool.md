---
title: Apktoolの使い方(コマンドラインオプションの解説)
author: watiko
type: post
date: 2013-01-25T17:19:04+00:00
url: /2013/01/26/how-to-use-apktool/
pdrp_attributionLocation:
  - end
categories:
  - Android
  - Tool
tags:
  - Android
  - apktool

---

apktoolを導入してオプションなしで起動するとヘルプが表示されます。使い方が表示されるので1度目を通しておいた方がよいのですが英語となると尻込みしてしまう人が多いと思います。

そこで、この記事では日本語で解説(単に訳しただけとも言う；)することを目標にしています。

[tip]この記事ではapktoolは2013/01/12時点での最新版である1.5.1を導入済みであるとして解説していきます。[/tip]

まず事前知識としてですが、使い方で「<strong>[]</strong>」に囲まれている部分は省略することができます。
また「<strong>|</strong>」はORを意味します。（どちらかということを意味します。）

<!--more-->

```
Apktool v1.5.1 - a tool for reengineering Android apk files
Copyright 2010 Ryszard Wi?niewski <アドレス削除>
with smali v1.4.1, and baksmali v1.4.1
Updated by @iBotPeaches <アドレス削除>
Apache License 2.0 (http://www.apache.org/licenses/LICENSE-2.0)

使い方: apktool [-q|--quiet あるいは -v|--verbose] コマンド [オプション]
    -q, --quiet
        詳細を表示しない。
    -v, --verbose
        詳細を表示する。（デフォルトで有効）

コマンド:

    d[ecode] [オプション] <対象ファイル名> [デコードしたものを展開するフォルダ]
        展開するフォルダを省略すると、対象ファイル名でフォルダが作成される。

        オプション:

        -s, --no-src
            ソース(classes.dex)をデコードしない。
        -r, --no-res
            リソース(resources.arsc,諸々のxml等)をデコードしない。
        -d, --debug
            デバッグモードでデコード、詳しくはGoogleCode上のプロジェクトを参照。←恐らく<a title="SmaliDebugging" href="https://code.google.com/p/android-apktool/wiki/SmaliDebugging" target="_blank">このページ</a>
        -b, --no-debug-info
            Baksmaliはデコードしたsmaliにデバッグ情報を含めない。 (.local, .param, .line, etc.)
        -f, --force
            対象フォルダを上書き。
        -t <tag>, --frame-tag <tag>
            <tag>でタグ付けされたリソースを参照する。
        --frame-path <dir>
            特定のフォルダのフレームワークファイルを参照する。
        --keep-broken-res
            エラーが表示されリソースが欠損する場合、(例えば「Invalid config flags detected. Dropping resources」等)
            それでも全てをデコードしたい時に用る。但し、ビルドする前にエラーの原因を見つけて修正する必要がある。

    b[uild] [オプション] [デコード済みのフォルダ名] [作成するファイル名]
        デコード済みフォルダのアプリをビルドします。

        自動でファイルが変更されたかを検出し、必要な手順のみ実行する。

        フォルダ名が省略された場合はカレントディレクトリが指定されたと見なされます。
        ファイル名が省略された場合は「フォルダ名/dist/オリジナルファイル名」が指定されたと見なされる。

        オプション:

        -f, --force-all
            ファイルの変更を検出せず、全てのファイルをビルドする。
        -d, --debug
            デバッグモードでビルド、詳しくはGoogleCode上のプロジェクトを参照。←恐ら<a title="SmaliDebugging" href="https://code.google.com/p/android-apktool/wiki/SmaliDebugging" target="_blank">このページ</a>
        -a, --aapt
            指定した場所のaaptを用いる。

        if|install-framework <framework.apk> [<tag>]
            フレームワークファイルをシステムにインストールします。
            <tag>を用いると、デコードするときに特定のフレームワークを使用することができます。

For additional info, see: http://code.google.com/p/android-apktool/
For smali/baksmali info, see: http://code.google.com/p/smali/
```

基本的な使い方は

```sh
$ apktool d <app_path>
$ apktool b <app_path> <file_b.apk>;
$ sign <file_b.apk> // この行は後で解説します
```
signに関してはパスが通ったフォルダに置かれたバッチファイルです。
apktoolやzipalignも同じ場所に置いています。

ついでなのでこの記事で紹介しておきます。(署名に関して別記事を作る予定なので、その際はこの件は移動します。)  
apkファイルをインストールするには署名が必要なのでそれを簡単なコマンドで実行できるようにしています。  
署名に使うツールの準備や鍵の作成は事前に行っておく必要があります。（現状それだけを扱った記事がないので
<a href="http://watiko.net/2013/01/19/how-to-make-pobox5-4-work-in-any-devices-without-rooting/" title="POBox 5.4をXperia以外の端末でも動くようにする。(root不要)">この記事</a>の「apkのビルド、署名」のあたりを参考にしてください。
`sign.bat`

```
@echo off
:sign filename.apk //この形で用います。因みに引数はパスでも大丈夫です。
:するとカレントディレクトリにfilename_aligned.apkという署名・最適化済みのapkファイルが作成されます。
:注意点は引数に取ったファイルにも署名が上書きされてしまうところです。cpとか使えば回避可能。

REM zip.exeを同じフォルダに置いています。既存の署名を削除。
zip -d -q "%~dpn1%~x1" META-INF/*
REM test.keystoreを同じフォルダに置いています。次の行の＜パスワード＞は鍵を作成したときのパスワードと置き換えます。
jarsigner -sigalg SHA1withRSA -digestalg SHA1 -storepass ＜パスワード＞ -keystore "%~dp0\test.keystore" -keypass ＜パスワード＞ "%~dpn1%~x1" testkey
zipalign 4 "%~dpn1%~x1" "%cd%\%~n1_aligned%~x1"
echo Done.
```

オプションに関する細かな解説をしていませんが、とりあえず公開しておきます。
