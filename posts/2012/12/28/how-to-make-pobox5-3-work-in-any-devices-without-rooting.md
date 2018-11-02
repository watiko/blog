---
title: POBox 5.3をXperia以外の端末でも動くようにする。(root不要)
author: watiko
type: post
date: 2012-12-27T17:33:38+00:00
url: /2012/12/28/how-to-make-pobox5-3-work-in-any-devices-without-rooting/
featured_image: /wp-content/uploads/2012/12/Enabled-50letters-HandwritingFading.png
pdrp_attributionLocation:
  - end
categories:
  - Android
  - Application
  - IME
  - Modding
tags:
  - adb
  - Android
  - apktool
  - cmd
  - Grep
  - jdk
  - POBox
  - Smali

---
[<img class="alignnone size-large wp-image-26" alt="POBox 5.3で五十音キーボードと手書きの軌跡を有効化し、ソニーチェックを無効化した状態。" src="/image/2012/12/Enabled-50letters-HandwritingFading-1024x576.png" width="550" height="309" srcset="/image/2012/12/Enabled-50letters-HandwritingFading-1024x576.png 1024w, /image/2012/12/Enabled-50letters-HandwritingFading-300x168.png 300w, /image/2012/12/Enabled-50letters-HandwritingFading.png 1280w" sizes="(max-width: 550px) 100vw, 550px" />][1]

すでに[某所][2]で公開しましたが、一応。ちなみにWindows向けです。

POBox 5.3はAndroid 4.0/ICS 以上向けです。それ以下の端末では動作しません。（2.3/GB,2.2/CCなど）

[tip][この記事][3]でPOBox5.4について扱っています。タブレット向けレイアウトも追加されていますが、<span style="background-color: #ffff00;">個人的</span>にはタブレット向けにはPOBox5.3の方がいいと思います。[/tip]

<!--more-->

準備するもの（ツール編）:

  1. **やる気**（結構大事。
  2. _いろいろやる知識ちょこっと。_
  3. ****Grep**できるもの(ここでは[Grep and Replace][4]を使います。)**
  4. **テキストエディタ**（TeraPadでもサクラエディタでもEmEditorでもUTF-8対応のものならおｋ。ただし、Grep and Replaceを使う場合は不要。）
  5. **展開ソフト**(7-zipやWinRARなど。)
  6. **apktool**([android-apktool | Google Project Hosting][5])
  7. **JarSigner**
  8. **zipalign**（某所ではやり忘れてました；）
  9. **JDK**([英Oracle][6]、パスの設定は<a title="PATHの設定及び環境変数JAVA_HOMEの設定 - Javaダウンロードとインストール" href="http://www.javadrive.jp/install/jdk/index4.html" rel="nofollow">ここ</a>を参考にしてみてください。一応パックにはバッチを含めてあります。)

<p style="padding-left: 60px;">
  3,4,5,9については各自インストールしておいてください。
</p>

<p style="padding-left: 60px;">
  6,7,8については別途記事を書きます。たぶん。とりあえず<a title="APK-modding-tools.zip" href="http://www.mediafire.com/?vs8sdu35sy7dd46">パック</a><span style="color: #ff0000;">(#2012/01/10 23:10更新)</span>しておきました（Windows向け）。
</p>

<p style="padding-left: 60px;">
  あるいはAPK Multi-Toolを使うと6,7,8は簡略化できます。
</p>

&nbsp;

準備するもの（データ編）:

  1. POBox5.3関係のファイル（[POBox 5.3 | 所感 ~android~][7]）
  2. apkを展開（デコード）するのに必要なファイル([SemcGenericUxpResVL.apk][8])

<!--more-->

手順:

# １．適当なフォルダを作成し、コマンドプロンプトで開く。

<p style="padding-left: 30px;">
  適当なフォルダを今回はC:\Temp\POBoxとします。必要に応じて各自読み替えても結構です。
</p>

<p style="padding-left: 30px;">
  ついでにいろいろ展開。
</p>

<p style="padding-left: 30px;">
  コマンドプロンプトやGrep and Replaceなどは指示があるまで終了しないでください。
</p>

「**Windowsキー+R**」を押下しファイル名を指定して実行を開き「**cmd**」入力しOKをクリック。

[<img class="alignnone size-full wp-image-28" alt="ファイル名を指定して実行" src="/image/2012/12/f79cca1cca4ccd85f013075d15bad303.png" width="413" height="274" srcset="/image/2012/12/f79cca1cca4ccd85f013075d15bad303.png 413w, /image/2012/12/f79cca1cca4ccd85f013075d15bad303-300x199.png 300w" sizes="(max-width: 413px) 100vw, 413px" />][9]

すると

[<img class="alignnone size-full wp-image-29" style="max-width: 100%; height: auto;" alt="コマンドプロンプト" src="/image/2012/12/c895fa206e3fcd705d0b51efa22d0bf5.png" srcset="/image/2012/12/c895fa206e3fcd705d0b51efa22d0bf5.png 629w, /image/2012/12/c895fa206e3fcd705d0b51efa22d0bf5-300x81.png 300w" sizes="(max-width: 629px) 100vw, 629px" />][10]

こんなんが出てくるので、

<pre>mkdir C:\Temp\POBox
C:
cd C:\Temp\POBox
start .</pre>

この4行をコピペして「**Enter**」を押すとこんな感じになると思います。

[<img class="alignnone size-full wp-image-30" style="max-width: 100%; height: auto;" alt="コピペ後のコマンドプロンプト" src="/image/2012/12/7a729c2718237dd6b3d82e9a44224ec1.png" srcset="/image/2012/12/7a729c2718237dd6b3d82e9a44224ec1.png 851w, /image/2012/12/7a729c2718237dd6b3d82e9a44224ec1-300x205.png 300w" sizes="(max-width: 767px) 89vw, (max-width: 1000px) 54vw, (max-width: 1071px) 543px, 580px" />][11]

エクスプローラーで「C:\Temp\POBox」が開かれたはずなのでそこに先ほどダウンロードした、APK-modding-tools.zipを展開。

続いて、POBox_5.3cwm.zipを7zipやWinRARなどで開きsystemの下のlibというフォルダ、appフォルダの中身も展開。<figure id="attachment_32" style="width: 256px" class="wp-caption alignnone">

<img class="alignnone size-full wp-image-45" alt="手順1の段階でのフォルダの中身" src="/image/2012/12/c63541f6febf5c3013744be3ccb301a01.png" width="256" height="220" /><figcaption class="wp-caption-text">手順1の段階でのフォルダの中身  
最新のzipではzip.exeとtr.exe、set_java.batが加わっています。</figcaption></figure> 

&nbsp;

&nbsp;

#  ２．展開してlibを配置、文字列の置換

何も言わずにコマンドプロンプトに以下4行をコピペして「**Enter**」

<pre>zip -d POBoxSknSelector.apk META-INF/*
apktool if SemcGenericUxpResVL.apk Sony2012
apktool d -b -r -t Sony2012 JapaneseIME.apk POBox53
xcopy C:\Temp\POBox\lib C:\Temp\POBox\POBox53\lib\armeabi\</pre>

その後、Grep and Replaceを開き

ファイルの名前に「*」、

含まれる文字列に「/system/lib/」、

探す場所に「C:\Temp\POBox\POBox53\lib\armeabi\」、

置換に「/data/data/com.sonyericsson.android.pobox/lib/」と入力します。

また、動作設定をクリックし、「保存時バックアップ作成」を無効化します。

&nbsp;<figure id="attachment_33" style="width: 603px" class="wp-caption alignnone">

[<img class="size-full wp-image-33 " alt="Grep and Replace" src="/image/2012/12/Grep-and-Replace.png" width="603" height="467" srcset="/image/2012/12/Grep-and-Replace.png 603w, /image/2012/12/Grep-and-Replace-300x232.png 300w" sizes="(max-width: 603px) 100vw, 603px" />][12]<figcaption class="wp-caption-text">入力が終わった状態。</figcaption></figure> 

次に、「検索」をクリックすると「置換」が押せるようになるので押します。<figure id="attachment_34" style="width: 602px" class="wp-caption alignnone">

[<img class="size-full wp-image-34 " alt="Grep and Replace" src="/image/2012/12/Grep-and-Replace2.png" width="602" height="467" srcset="/image/2012/12/Grep-and-Replace2.png 602w, /image/2012/12/Grep-and-Replace2-300x232.png 300w" sizes="(max-width: 602px) 100vw, 602px" />][13]<figcaption class="wp-caption-text">追記した手順を行うと、IWnnEngine.smali、IMorphemeService.smaliがリストアップされます。</figcaption></figure> 

すると、このようになるので、「リストのファイルを一括置換」を選択。

[<img class="alignnone size-full wp-image-35" alt="Grep and Replace" src="/image/2012/12/Grep-and-Replace3.png" width="483" height="232" srcset="/image/2012/12/Grep-and-Replace3.png 483w, /image/2012/12/Grep-and-Replace3-300x144.png 300w" sizes="(max-width: 483px) 100vw, 483px" />][14]

&nbsp;

保存確認が3回出るので全部「はい」を選択します。

&nbsp;

<span style="color: #ff0000;">追記</span>: 実行する際は最新のパックを用いてください。

左下の検索設定をクリックし、「サブフォルダも探す」にチェックを入れ有効化します。

その後、探す場所に「C:\Temp\POBox\POBox53\smali」を入れ再度検索、置換、リストのファイルを一括置換を選択します。（暫く時間がかかると思います。）

また、改行コードを修正するために次のコードをコマンドプロンプトにコピペし「**Enter**」

<pre>set libPath=C:\Temp\POBox\POBox53\lib\armeabi
tr -dIO \r &lt; %libPath%\lib_dic_en_USUK.conf.so &gt; %libPath%\lib_dic_en_USUK.conf.tmp
tr -dIO \r &lt; %libPath%\lib_dic_ja_JP.conf.so &gt; %libPath%\lib_dic_ja_JP.conf.tmp
tr -dIO \r &lt; %libPath%\lib_dic_morphem_ja_JP.conf.so &gt; %libPath%\lib_dic_morphem_ja_JP.conf.tmp
move /y %libPath%\lib_dic_en_USUK.conf.tmp %libPath%\lib_dic_en_USUK.conf.so
move /y %libPath%\lib_dic_ja_JP.conf.tmp %libPath%\lib_dic_ja_JP.conf.so
move /y %libPath%\lib_dic_morphem_ja_JP.conf.tmp %libPath%\lib_dic_morphem_ja_JP.conf.so</pre>

&nbsp;

&nbsp;

# ３．smaliの書き換え

<p style="padding-left: 30px;">
  ここでもGrep and Replaceを使います。
</p>

<p style="padding-left: 30px;">
  探す場所に「C:\Temp\POBox\POBox53\smali」と入力しておいてください。
</p>

<p style="padding-left: 30px;">
  引数はやるたびに違うので参考程度にとどめてください。
</p>

## ・ソニーチェックの無効化

該当ファイル：

  * OpenWnn.smali
  * IWnnLanguageSwitcher.smali
  * ControlPanelJaJp.smali

含まれる文字列に「Ljava/lang/String;->startsWith(Ljava/lang/String;)Z」を入力して検索。

その後、上記のファイル名をクリックするとウィンドウの右下に内容が表示されるため、赤い下線が引かれた行の引数(ひきすう)を一致させます。この際、どちらに合わせてもよいのですが、ここでは後ろの引数に合わせます。1つのファイルが終わり次のファイルを選ぶと保存確認が出るのでOKを選んでください。3つめが終わったあとは保存するためどれかファイル名をクリックしてください。

[<img class="alignnone size-full wp-image-37" style="max-width: 100%; height: auto;" alt="Grep and Replace" src="/image/2012/12/Grep-and-Replace5.png" srcset="/image/2012/12/Grep-and-Replace5.png 913w, /image/2012/12/Grep-and-Replace5-300x195.png 300w" sizes="(max-width: 767px) 89vw, (max-width: 1000px) 54vw, (max-width: 1071px) 543px, 580px" />][15]

&nbsp;

## ・50音キーボードと手書きの軌跡有効化

該当ファイル：

  * POBoxConfig.smali

Lcom/sonyericsson/android/pobox/preferences/POBoxPreference;->setEnable50Letter(ZZ)V
  
Lcom/sonyericsson/android/pobox/preferences/POBoxPreference;->setEnableHandwritingFading(ZZ)V

この2行には2つではなく3つの引数が含まれるので後半の2つを一致させてください。

たとえば、

<pre>invoke-virtual {v1, v0, v2}, Lcom/sonyericsson/android/pobox/preferences/POBoxPreference;-&gt;setEnable50Letter(ZZ)V</pre>

これを

<pre>invoke-virtual {v1, v2, v2}, Lcom/sonyericsson/android/pobox/preferences/POBoxPreference;-&gt;setEnable50Letter(ZZ)V</pre>

こう。

&nbsp;

## ・存在しないリソースへの参照を書き換え

該当ファイル、書き換え回数:

  * IWnnImeJaJp.smali ×2
  * SkinData.smali ×3

含まれる文字列で「

<pre>0x206</pre>

」を検索します。

IWnnImeJaJp.smaliの場合は、

<pre>invoke-virtual {p0}, Lcom/sonyericsson/android/pobox/core/IWnnImeJaJp;-&gt;getResources()Landroid/content/res/Resources;

    move-result-object v5

    const/high16 v6, 0x206

    invoke-virtual {v5, v6}, Landroid/content/res/Resources;-&gt;getColor(I)I

    move-result v5</pre>

この「invoke-direct」から「move-result v5」までを次の1行に置き換えます。

<pre>const/16 v5, 0xffffffff</pre>

このとき「move-result v5」の「v5」という引数と、置き換えたあと「const/16 v5, 0xffffffff」の「v5」という引数が一致することに注意してください。これを2ヶ所。

&nbsp;

** 2013/01/06追記**

<del datetime="2013-01-07T05:38:07+00:00">コメントからこうならない場合も有ると発覚したので上記のようなヶ所が見当たらなかった場合は<a href="#comment-7">コメント</a>も見てみてください。</del>
  
手順を確認していて勘違いに気づきました。間違った修正箇所を掲載していたようです。すみません。したがって次の画像の書き換え範囲は間違っています。

&nbsp;

[<img class="alignnone size-full wp-image-38" style="max-width: 100%; height: auto;" alt="Grep and Replace" src="/image/2012/12/Grep-and-Replace6.png" srcset="/image/2012/12/Grep-and-Replace6.png 913w, /image/2012/12/Grep-and-Replace6-300x195.png 300w" sizes="(max-width: 767px) 89vw, (max-width: 1000px) 54vw, (max-width: 1071px) 543px, 580px" />][16]

&nbsp;

SkinData.smaliの場合は、

<pre>const/high16 v2, 0x206

    invoke-virtual {p1, v2}, Landroid/content/res/Resources;-&gt;getColor(I)I

    move-result v0</pre>

これを、次の1行に置き換えます。

<pre>const/16 v0, 0xffffffff</pre>

先ほどと同じで、引数を一致させるように気をつけてください。3ヶ所。

&nbsp;

## ・存在しないフォントの置き換え

該当ファイル:

  * KeyboardView.smali

SoMARegular.ttf→Roboto-Regular.ttf

SoMABold.ttf→Roboto-Bold.ttf

このようになるように置き換えます。それぞれ2ヶ所の計四ヶ所。

&nbsp;

#  ４．apkのビルド、署名

<p style="padding-left: 30px;">
   Grep and Replaceはこの段階になったら終了してください。
</p>

## ・apkのビルド

コマンドプロンプトに次のコマンドをコピペし「**Enter**」

<pre>apktool b POBox53 POBox5.3.apk</pre>

##  ・apkの署名

<p style="padding-left: 30px;">
  この作業はとても厄介なのでしっかりと確認してください。
</p>

コマンドプロンプトに次の行をコピペして「**Enter**」

<pre>keytool -genkey -v -sigalg SHA1withRSA -keyalg RSA -keystore test.keystore -alias testkey -validity 10000</pre>

すると、キーストアのパスワード、姓名、組織名、都市名、州名、国番号を問われるので答えていきます。答えを入力、「**Enter**」という流れです。

<p style="padding-left: 30px;">
  パスワード入力中は入力中の文字は表示されません。
</p>

<p style="padding-left: 30px;">
  必要になった際に使えるようにしておくためキーストアパスワードは忘れないでください。
</p>

国番号を答えたあと、 「[いいえ]:」とひょうじされるので「y」と入力し「**Enter**」

その次に再度「**Enter**」と押すと署名に必要なキーができあがります。

&nbsp;

コマンドプロンプトに次の行をコピペして「**Enter**」

<pre>jarsigner -sigalg SHA1withRSA -digestalg SHA1 -keystore test.keystore -verbose POBox5.3.apk testkey</pre>

キーストアのパスワードを聞かれるので入力して「**Enter**」

&nbsp;

次の行も同じです。

<pre>jarsigner -sigalg SHA1withRSA -digestalg SHA1 -keystore test.keystore -verbose POBoxSknSelector.apk testkey</pre>

&nbsp;

## ・apkの最適化(zipalign)

次の2行をコピペして「**Enter**」

<pre>zipalign -v 4 POBox5.3.apk POBox5.3_aligned.apk
zipalign -v 4 POBoxSknSelector.apk POBoxSknSelector_aligned.apk</pre>

&nbsp;

#  ５．インストール

<p style="padding-left: 30px;">
  さすがにここは書かなくてもいいかなと思ったのですが一応書いておきます。
</p>

ここまででapkの改変そのものは作業が終了しています。

改変済みなのとインストールした方がよいapkは

  * POBox5.3_aligned.apk
  * POBoxSknSelector_aligned.apk
  * POBoxSknMono.apk

&nbsp;

## ・提供元不明のアプリ

Android端末の設定を開きます。

セキュリティをタップし、提供元不明のアプリにチェックを入れます。すると警告が表示されるのでしっかりと読んだ上で「OK」をタップします。<figure id="attachment_48" style="width: 360px" class="wp-caption alignnone">

[<img class="size-full wp-image-48" alt="提供元不明のアプリ" src="/image/2012/12/d7abb477ad1bb4688dd2a75134351136.png" width="360" height="640" srcset="/image/2012/12/d7abb477ad1bb4688dd2a75134351136.png 720w, /image/2012/12/d7abb477ad1bb4688dd2a75134351136-168x300.png 168w, /image/2012/12/d7abb477ad1bb4688dd2a75134351136-576x1024.png 576w" sizes="(max-width: 360px) 100vw, 360px" />][17]<figcaption class="wp-caption-text">TabUIなので通常の表示とは違いますがこんな感じです。</figcaption></figure> 

&nbsp;

## ・インストール

adbが使える環境なら話は簡単で、端末をPCにつなぎ次の3行をコマンドプロンプトにコピペ、「**Enter**」で完了です。

<pre>adb install POBoxSknSelector_aligned.apk
adb install POBoxSknMono.apk
adb install POBox5.3_aligned.apk</pre>

問題はそういった環境がない場合です。

たとえばPCと端末をつなぎ、SDカード上に「apk」など適当なフォルダを作り、前述のapkを転送します。

そこに「<a title="ES File Explorer | Google Play" href="https://play.google.com/store/apps/details?id=com.estrongs.android.pop" target="_blank">ES ファイルエクスプローラー</a>」などを使ってアクセスし、インストールしたいapkをタップすればインストール可能です。

&nbsp;

## ・有効化

ほかのIMEを使ったことがある人は知っていると思いますが、POBoxもその例に外れず、インストールしただけでは使うことができません。

そこで何をすればよいかというと、先ほどのようにAndroidの設定を開きます。

言語と入力をタップし、POBox Touch (日本語)にチェックを入れます。ここでも注意が表示されるので納得の上で「OK」をタップしてください。<figure id="attachment_49" style="width: 360px" class="wp-caption alignnone">

[<img class="size-full wp-image-49" alt="言語と入力" src="/image/2012/12/52c56a315700b05470164497f459245e.png" width="360" height="640" srcset="/image/2012/12/52c56a315700b05470164497f459245e.png 720w, /image/2012/12/52c56a315700b05470164497f459245e-168x300.png 168w, /image/2012/12/52c56a315700b05470164497f459245e-576x1024.png 576w" sizes="(max-width: 360px) 100vw, 360px" />][18]<figcaption class="wp-caption-text">IMEが多すぎる件にはあんまり突っ込まないでください…</figcaption></figure> 

&nbsp;

##  ・テスト

無事に起動しましたか？50音キーボードの有効化はできていますか？手書きの軌跡は表示されていますか？

何事もなく成功した人はお疲れ様でした。

失敗してしまった人はトライアンドエラーを繰り返してみてください。それでもだめなら、分かる範囲、できる範囲で対応します。

&nbsp;

&nbsp;

# ６．最後に

ここまで読んでくれてありがとうございました。

初めて書いた記事なので至らない点など多々あると思いますが、なにとぞご容赦のほどを…

[Twitter -- @watiko][19]

&nbsp;

追記:2013/01/07 22:40

XP Mode上で一通り動作するのを確認しました。一先ず、手順はこれでよいはずです。

&nbsp;

&nbsp;

<span style="color: #c0c0c0;">一応今回できたもの<a title="POBox5.3_aligned.rar" href="http://www.mediafire.com/?znxcokrss5yh13c"><span style="color: #c0c0c0;">POBox5.3</span></a></span>

&nbsp;

では。

 [1]: /image/2012/12/Enabled-50letters-HandwritingFading.png
 [2]: http://pastebin.com/CFWfwhAR "Pastebin | Po53"
 [3]: http://watiko.net/2013/01/19/how-to-make-pobox5-4-work-in-any-devices-without-rooting/ "POBox 5.4をXperia以外の端末でも動くようにする。(root不要)TypoTypoTypo! | TypoTypoTypo!"
 [4]: http://www.vector.co.jp/soft/dl/win95/util/se205255.html "ファイルの検索・一括置換 Grep and Replaceのダウンロード : Vector ソフトを探す！"
 [5]: https://code.google.com/p/android-apktool/downloads/list "android-apktool |Google Project Hosting"
 [6]: http://www.oracle.com/technetwork/java/javase/downloads/index.html "JavaSEのダウンロード"
 [7]: http://thjap.org/android/xperia-series/783.html "POBox 5.3 | 所感 ~android~"
 [8]: http://www.mediafire.com/?ms7y26wi36ie417 "SemcGenericUxpResVL.apk"
 [9]: /image/2012/12/f79cca1cca4ccd85f013075d15bad303.png
 [10]: /image/2012/12/c895fa206e3fcd705d0b51efa22d0bf5.png
 [11]: /image/2012/12/7a729c2718237dd6b3d82e9a44224ec1.png
 [12]: /image/2012/12/Grep-and-Replace.png
 [13]: /image/2012/12/Grep-and-Replace2.png
 [14]: /image/2012/12/Grep-and-Replace3.png
 [15]: /image/2012/12/Grep-and-Replace5.png
 [16]: /image/2012/12/Grep-and-Replace6.png
 [17]: /image/2012/12/d7abb477ad1bb4688dd2a75134351136.png
 [18]: /image/2012/12/52c56a315700b05470164497f459245e.png
 [19]: https://twitter.com/watiko "@watiko"