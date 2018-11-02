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
[markdown]
  
apktoolを導入してオプションなしで起動するとヘルプが表示されます。使い方が表示されるので1度目を通しておいた方がよいのですが英語となると尻込みしてしまう人が多いと思います。

そこで、この記事では日本語で解説(単に訳しただけとも言う；)することを目標にしています。

[tip]この記事ではapktoolは2013/01/12時点での最新版である1.5.1を導入済みであるとして解説していきます。[/tip]

まず事前知識としてですが、使い方で「\*\*[]\*\*」に囲まれている部分は省略することができます。
  
また「\*\*|\*\*」はORを意味します。（どちらかということを意味します。）

<!--more-->

&#8220;\`sh
  
Apktool v1.5.1 &#8211; a tool for reengineering Android apk files
  
Copyright 2010 Ryszard Wi?niewski <アドレス削除>
  
with smali v1.4.1, and baksmali v1.4.1
  
Updated by @iBotPeaches <アドレス削除>
  
Apache License 2.0 (http://www.apache.org/licenses/LICENSE-2.0)

使い方: apktool [-q|&#8211;quiet あるいは -v|&#8211;verbose] コマンド [オプション]
      
-q, &#8211;quiet
          
詳細を表示しない。
      
-v, &#8211;verbose
          
詳細を表示する。(デフォルトで有効)

コマンド:

d\[ecode\] \[オプション\] <対象ファイル名> [デコードしたものを展開するフォルダ]
          
展開するフォルダを省略すると、対象ファイル名でフォルダが作成される。

オプション:

-s, &#8211;no-src
              
ソース(classes.dex)をデコードしない。
          
-r, &#8211;no-res
              
リソース(resources.arsc,諸々のxml等)をデコードしない。
          
-d, &#8211;debug
              
デバッグモードでデコード、詳しくはGoogleCode上のプロジェクトを参照。←恐らくこのページ https://code.google.com/p/android-apktool/wiki/SmaliDebugging
          
-b, &#8211;no-debug-info
              
Baksmaliはデコードしたsmaliにデバッグ情報を含めない。 (.local, .param, .line, etc.)
          
-f, &#8211;force
              
対象フォルダを上書き。
          
-t <tag>, &#8211;frame-tag <tag>
              
<tag>でタグ付けされたリソースを参照する。
          
&#8211;frame-path 

<dir>
  <br /> 特定のフォルダのフレームワークファイルを参照する。<br /> &#8211;keep-broken-res<br /> エラーが表示されリソースが欠損する場合、(例えば「Invalid config flags detected. Dropping resources」等)<br /> それでも全てをデコードしたい時に用る。但し、ビルドする前にエラーの原因を見つけて修正する必要がある。</p> 
  
  <p>
    b[uild] [オプション] [デコード済みのフォルダ名] [作成するファイル名]<br /> デコード済みフォルダのアプリをビルドします。
  </p>
  
  <p>
    自動でファイルが変更されたかを検出し、必要な手順のみ実行する。
  </p>
  
  <p>
    フォルダ名が省略された場合はカレントディレクトリが指定されたと見なされます。<br /> ファイル名が省略された場合は「フォルダ名/dist/オリジナルファイル名」が指定されたと見なされる。
  </p>
  
  <p>
    オプション:
  </p>
  
  <p>
    -f, &#8211;force-all<br /> ファイルの変更を検出せず、全てのファイルをビルドする。<br /> -d, &#8211;debug<br /> デバッグモードでビルド、詳しくはGoogleCode上のプロジェクトを参照。←恐らくこのページ https://code.google.com/p/android-apktool/wiki/SmaliDebugging<br /> -a, &#8211;aapt<br /> 指定した場所のaaptを用いる。
  </p>
  
  <p>
    if|install-framework <framework.apk> [<tag>]<br /> フレームワークファイルをシステムにインストールします。<br /> <tag>を用いると、デコードするときに特定のフレームワークを使用することができます。
  </p>
  
  <p>
    For additional info, see: http://code.google.com/p/android-apktool/<br /> For smali/baksmali info, see: http://code.google.com/p/smali/<br /> &#8220;`
  </p>
  
  <p>
    基本的な使い方は
  </p>
  
  <p>
    &#8220;`sh<br /> apktool d <app_path><br /> apktool b <app_path> <file_b.apk><br /> sign <file_b.apk> //この行は後で解説します。<br /> &#8220;`
  </p>
  
  <p>
    signに関してはパスが通ったフォルダに置かれたバッチファイルです。<br /> apktoolやzipalignも同じ場所に置いています。
  </p>
  
  <p>
    ついでなのでこの記事で紹介しておきます。(署名に関して別記事を作る予定なので、その際はこの件は移動します。)<br /> apkファイルをインストールするには署名が必要なのでそれを簡単なコマンドで実行できるようにしています。<br /> 署名に使うツールの準備や鍵の作成は事前に行っておく必要があります。（現状それだけを扱った記事がないので<a href="http://watiko.net/2013/01/19/how-to-make-pobox5-4-work-in-any-devices-without-rooting/" title="POBox 5.4をXperia以外の端末でも動くようにする。(root不要)">この記事</a>の「apkのビルド、署名」のあたりを参考にしてください。
  </p>
  
  <p>
    ###sign.bat
  </p>
  
  <p>
    &#8220;`sh<br /> @echo off<br /> :sign filename.apk //この形で用います。因みに引数はパスでも大丈夫です。<br /> :するとカレントディレクトリにfilename_aligned.apkという署名・最適化済みのapkファイルが作成されます。<br /> :注意点は引数に取ったファイルにも署名が上書きされてしまうところです。cpとか使えば回避可能。
  </p>
  
  <p>
    REM zip.exeを同じフォルダに置いています。既存の署名を削除。<br /> zip -d -q &#8220;%~dpn1%~x1&#8221; META-INF/*<br /> REM test.keystoreを同じフォルダに置いています。次の行の＜パスワード＞は鍵を作成したときのパスワードと置き換えます。<br /> jarsigner -sigalg SHA1withRSA -digestalg SHA1 -storepass ＜パスワード＞ -keystore &#8220;%~dp0\test.keystore&#8221; -keypass ＜パスワード＞ &#8220;%~dpn1%~x1&#8221; testkey<br /> zipalign 4 &#8220;%~dpn1%~x1&#8221; &#8220;%cd%\%~n1_aligned%~x1&#8221;<br /> echo Done.<br /> &#8220;`
  </p>
  
  <p>
    オプションに関する細かな解説をしていませんが、とりあえず公開しておきます。<br /> 分からないところがあればコメントで。<br /> [/markdown]
  </p>