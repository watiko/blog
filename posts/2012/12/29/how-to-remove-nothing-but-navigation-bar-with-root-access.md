---
title: ステータスバーはそのままにナビゲーションバーだけを消す方法（要root、カスタムROM不要）
author: watiko
type: post
date: 2012-12-29T14:02:34+00:00
url: /2012/12/29/how-to-remove-nothing-but-navigation-bar-with-root-access/
pdrp_attributionLocation:
  - end
categories:
  - Android
  - Modding
  - root
tags:
  - adb
  - Android
  - apktool
  - system framework

---
タイトルの通りですが、試して見ました。

System領域をいじるので文鎮()状態になる危険はありますが復旧は可能です。

（完全に文鎮になることはないと思います。たぶん。）

また、作業を行う際は自己責任でお願いします。

あ、あと基本的にはwindows向けの記事です。

<!--more-->

用意するもの:

  * [apktool 1.51以上 ][1]
  * Java SE (JRE) ※JDK導入済みなら必要なし。
  * Unicode(UTF-8)が扱えるテキストエディタ
  * 7-zipやWinRARなど展開ソフト
  * adb導入済みの環境
  * 当然ながらAndroid 4.0以上のrooted端末

[warning]あらかじめナビゲーションバーの代わりに端末を操作するアプリケーションを導入の上で実行してください。[/warning]

手順:

１．適当なフォルダ(ここではC:\Temp\framework\とします。)を作りコマンドプロンプトで開く。

「**Win+R**」で開くファイル名を指定して実行に「cmd」と入力して「**Enter**」

コマンドプロンプトが開くので、以下のコマンドをコピペして「**Enter**」

[tip]コマンドをコピペしてから「**Enter**」という指示では、動作に時間がかかることがあります。「**Enter**」を押すタイミングは「>」の右下が点滅しているときです。[/tip]

<pre>mkdir C:\Temp\framework\
C:
cd C:\Temp\framework\
start .</pre>

このとき、C:\Temp\framework\がエクスプローラーで開かれます。

&nbsp;

２．端末からframework-res.apkを取り出し、改変する。

先ほど開いたコマンドプロンプトに以下のコマンドをコピペして「**Enter**」

<pre>adb pull /system/framework/framework-res.apk framework-res_mod.apk
apktool if framework-res_mod.apk
apktool d framework-res_mod.apk src</pre>

次に、C:\Temp\framework\src\res\values\bools.xmlをテキストエディタで開きます。

開いたファイルから

<pre>    &lt;bool name="config_showNavigationBar"&gt;true&lt;/bool&gt;</pre>

と書いてある行を探し、その行の>**true**<を>**false**<に書き換えます。

ファイルを保存し、コマンドプロンプトに以下のコマンドをコピペして「**Enter**」

<pre>apktool b src</pre>

すると、C:\Temp\framework\src.apkができるので7-zip(WinRAR)で開き「resources.arsc」をC:\Temp\framework\framework-res_mod.apkの「resources.arsc」に上書き。

（src.apkからframework-res_mod.apkに上書きです。）

&nbsp;

３．転送、配置

コマンドプロンプトに以下のコマンドをコピペして「**Enter**」

<pre>adb shell su -c "mount -o remount,rw /system"
adb shell mkdir /sdcard/Temp/
adb push framework-res_mod.apk /sdcard/Temp/
adb shell su -c "dd if=/system/framework/framework-res.apk of=/system/framework/framework-res.bak"
adb shell su -c "dd if=/sdcard/Temp/framework-res_mod.apk of=/system/framework/framework-res.apk"</pre>

「Enter」を押したあと3行ほど出力された後にすぐ再起動します。

起動後は既にナビゲーションバーは表示されていませんお疲れ様でした。

&nbsp;

４．後始末、復旧

目的は達成しましたが、後始末が終わっていないので次のコマンドをコピペして「**Enter**」

<pre>adb shell rm -r /sdcard/Temp</pre>

これで後始末まで終わりました。

仮に文鎮()状態になってしまったら、まずリカバリー経由でadb接続をします。その際、リカバリーからSystem領域をマウントしておいてください。そして次のコマンドをコピペして「**Enter**」

<pre>adb shell dd if=/system/framework/framework-res.bak of=/system/framework/framework-res.apk</pre>

本当にお疲れ様でした。

&nbsp;

では。

 [1]: https://code.google.com/p/android-apktool/downloads/list "Downloads - android-apktool - A tool for reverse engineering Android apk files - Google Project Hosting"