---
title: POBox 5.4をXperia以外の端末でも動くようにする。(root不要)
author: watiko
type: post
date: 2013-01-19T11:51:00+00:00
url: /2013/01/19/how-to-make-pobox5-4-work-in-any-devices-without-rooting/
featured_image: /wp-content/uploads/2013/01/Screenshot_2013-01-19-20-36-09.png
pdrp_attributionLocation:
  - end
categories:
  - Android
  - Application
  - IME
  - Modding
tags:
  - Android
  - apktool
  - POBox
  - Smali

---
[<img class="alignnone size-large wp-image-120" alt="POBox 5.4で五十音キーボードと手書きの軌跡を有効化し、ソニーチェックを無効化した状態。" src="/image/2013/01/Screenshot_2013-01-19-20-36-09-1024x576.png" width="550" height="309" srcset="/image/2013/01/Screenshot_2013-01-19-20-36-09-1024x576.png 1024w, /image/2013/01/Screenshot_2013-01-19-20-36-09-300x168.png 300w, /image/2013/01/Screenshot_2013-01-19-20-36-09.png 1280w" sizes="(max-width: 550px) 100vw, 550px" />][1]

POBox 5.4はAndroid 4.1/JB 以上向けです。それ以下の端末では動作しません。（2.3/GB,2.2/CCなど）

<span style="color: #c0c0c0;">ついでに言うと、POBoxがプリインストールの端末(Xperiaシリーズ)にも署名の関係でストレートにはインストール出来ません。残念ながらroot権限でシステムアプリとしてのJapaneseIME.apkとPOBoxSknSelector.apkを削除しておく必要がありますx(</span>

※ただし、4.0/ICSはある手順を踏めば導入可能です。これに関しては、やや面倒なのでこの記事では扱いませんコメント欄にメモ程度にかいておきます。

※サイズ変更時のバグは<span style="color: #ff0000;">解消</span>されています。

&nbsp;

前回、[POBox 5.3をXperia以外の端末でも動くようにする。(root不要) ][2]この記事でPOBox5.3の導入を行った人は一部作業が重複するので適宜対処してください。（C:\Temp\POBoxを削除、一部作業を飛ばす等）

<!--more-->

準備するもの（ツール編）:

  1. **やる気**（結構大事。
  2. _いろいろやる知識ちょこっと。_
  3. ****Grep**できるもの(ここでは[Grep and Replace][3]を使います。)**
  4. **テキストエディタ**（TeraPadでもサクラエディタでもEmEditorでもUTF-8対応のものならおｋ。ただし、Grep and Replaceを使う場合は不要。）
  5. **展開ソフト**(7-zipやWinRARなど。)
  6. **apktool**([android-apktool | Google Project Hosting][4])
  7. **JarSigner**
  8. **zipalign**（某所ではやり忘れてました；）
  9. **JDK**([英Oracle][5]、パスの設定は<a title="PATHの設定及び環境変数JAVA_HOMEの設定 - Javaダウンロードとインストール" href="http://www.javadrive.jp/install/jdk/index4.html" rel="nofollow">ここ</a>を参考にしてみてください。一応パックにはバッチを含めてあります。)

3,4,5,9については各自インストールしておいてください。

6,7,8については別途記事を書きます。たぶん。とりあえず[パック][6](#2012/01/10 23:10更新)しておきました（Windows向け）。

パックの中には手順に含まれるものもあるので導入はほぼ必須です。

&nbsp;

準備するもの（データ編）:

  1. POBox5.4関係のファイル（<a title="POBox5.4_ZD_src.zip" href="http://www.mediafire.com/?t9qjbjuj4s8z7y6" target="_blank">POBox5.4_ZD_src.zip</a>）

&nbsp;

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

[<img alt="ファイル名を指定して実行" src="/image/2012/12/f79cca1cca4ccd85f013075d15bad303.png" width="413" height="274" />][7]

すると

[![コマンドプロンプト][8]][8]

こんなんが出てくるので、

<pre>mkdir C:\Temp\POBox
C:
cd C:\Temp\POBox
start .</pre>

この4行をコピペして「**Enter**」を押すとこんな感じになると思います。

[![コピペ後のコマンドプロンプト][9]][9]

エクスプローラーで「C:\Temp\POBox」が開かれたはずなのでそこに先ほどダウンロードした、「APK-modding-tools.zip」と「POBox5.4_src.zip」を展開。<figure id="attachment_128" style="width: 203px" class="wp-caption alignnone">

[<img class="size-full wp-image-128" alt="aapt.exe apktool.bat apktool.jar JapaneseIME.apk lib POBoxSknMono.apk POBoxSknSelector.apk SemcGenericUxpResTJB.apk set_java.bat tr.exe zip.exe zipalign.exe" src="/image/2013/01/2013-01-19-20-59-09_POBox.png" width="203" height="263" />][10]<figcaption class="wp-caption-text">手順1の段階でのフォルダの中身</figcaption></figure> 

&nbsp;

# ２．展開してlibを配置、文字列の置換

何も言わずにコマンドプロンプトに以下4行をコピペして「**Enter**」

<pre>zip -d POBoxSknSelector.apk META-INF/*
apktool if SemcGenericUxpResZU.apk Sony_ZD
apktool d -b -r -t Sony_ZD JapaneseIME.apk POBox54
xcopy C:\Temp\POBox\lib C:\Temp\POBox\POBox54\lib\armeabi\</pre>

その後、Grep and Replaceを開き

ファイルの名前に「*」、

含まれる文字列に「/system/lib/」、

探す場所に「C:\Temp\POBox\POBox54\lib\armeabi\」、

置換に「/data/data/com.sonyericsson.android.pobox/lib/」と入力します。

また、動作設定をクリックし、「保存時バックアップ作成」を無効化します。<figure id="attachment_129" style="width: 603px" class="wp-caption alignnone">

[<img class="size-full wp-image-129" alt="Grep and Replace" src="/image/2013/01/2013-01-19-21-15-02_Grep-and-Replace.png" width="603" height="467" srcset="/image/2013/01/2013-01-19-21-15-02_Grep-and-Replace.png 603w, /image/2013/01/2013-01-19-21-15-02_Grep-and-Replace-300x232.png 300w" sizes="(max-width: 603px) 100vw, 603px" />][11]<figcaption class="wp-caption-text">入力が終わった状態</figcaption></figure> 

次に、「検索」をクリックすると「置換」が押せるようになるので押します。

[<img class="alignnone size-full wp-image-130" alt="Grep and Replace2" src="/image/2013/01/2013-01-19-21-15-02_Grep-and-Replace2.png" width="604" height="467" srcset="/image/2013/01/2013-01-19-21-15-02_Grep-and-Replace2.png 604w, /image/2013/01/2013-01-19-21-15-02_Grep-and-Replace2-300x231.png 300w" sizes="(max-width: 604px) 100vw, 604px" />][12]

&nbsp;

すると、このようになるので、「リストのファイルを一括置換」を選択。

[<img class="alignnone size-full wp-image-131" alt="Grep and Replace - 確認" src="/image/2013/01/6e5900aa07e7242384d66b0cd39613d6.png" width="375" height="214" srcset="/image/2013/01/6e5900aa07e7242384d66b0cd39613d6.png 375w, /image/2013/01/6e5900aa07e7242384d66b0cd39613d6-300x171.png 300w" sizes="(max-width: 375px) 100vw, 375px" />][13]

&nbsp;

「はい」を選択。

[<img class="alignnone size-full wp-image-132" alt="Grep and Replace - 保存確認" src="/image/2013/01/64dae904b8be20d3c1ab61ac86b74490.png" width="447" height="232" srcset="/image/2013/01/64dae904b8be20d3c1ab61ac86b74490.png 447w, /image/2013/01/64dae904b8be20d3c1ab61ac86b74490-300x155.png 300w" sizes="(max-width: 447px) 100vw, 447px" />][14]

&nbsp;

こんな感じのダイアログが3回出るので、それぞれ「はい」を選択。

&nbsp;

左下の検索設定をクリックし、「サブフォルダも探す」にチェックを入れ有効化します。

その後、探す場所に「C:\Temp\POBox\POBox54\smali」を入れ再度検索、置換、リストのファイルを一括置換を選択します。（暫く時間がかかると思います。）

また、改行コードを修正するために次のコードをコマンドプロンプトにコピペし「**Enter**」

<pre>set libPath=C:\Temp\POBox\POBox54\lib\armeabi
tr -dIO \r &lt; %libPath%\lib_dic_en_USUK.conf.so &gt; %libPath%\lib_dic_en_USUK.conf.tmp
tr -dIO \r &lt; %libPath%\lib_dic_ja_JP.conf.so &gt; %libPath%\lib_dic_ja_JP.conf.tmp
tr -dIO \r &lt; %libPath%\lib_dic_morphem_ja_JP.conf.so &gt; %libPath%\lib_dic_morphem_ja_JP.conf.tmp
move /y %libPath%\lib_dic_en_USUK.conf.tmp %libPath%\lib_dic_en_USUK.conf.so
move /y %libPath%\lib_dic_ja_JP.conf.tmp %libPath%\lib_dic_ja_JP.conf.so
move /y %libPath%\lib_dic_morphem_ja_JP.conf.tmp %libPath%\lib_dic_morphem_ja_JP.conf.so</pre>

&nbsp;

# ３．smaliの書き換え

<p style="padding-left: 30px;">
  ここでもGrep and Replaceを使います。
</p>

<p style="padding-left: 30px;">
  探す場所に「C:\Temp\POBox\POBox54\smali」と入力しておいてください。
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

その後、上記のファイル名をクリックするとウィンドウの右下に内容が表示されるため、赤い下線が引かれた行の引数(ひきすう)を一致させます。この際、どちらに合わせてもよいのですが、ここでは後ろの引数に合わせます。1つのファイルが終わり次のファイルを選ぶと保存確認が出るのでOKを選んでください。3つめが終わったあとは保存するためどれかファイル名をクリックしてください。<figure style="width: 913px" class="wp-caption alignnone">

[<img alt="Grep and Replace" src="/image/2012/12/Grep-and-Replace5.png" width="913" height="596" />][15]<figcaption class="wp-caption-text">画像は使い回しなので、フォルダ名などは読み替えてください。</figcaption></figure> 

&nbsp;

## ・50音キーボードと手書きの軌跡有効化

該当ファイル：

  * POBoxConfig.smali

Lcom/sonyericsson/android/pobox/preferences/POBoxPreference;->setEnable50Letter(ZZ)V
  
Lcom/sonyericsson/android/pobox/preferences/POBoxPreference;->setEnableHandwritingFading(ZZ)V

この2行には2つではなく3つの引数が含まれるので後半の2つを一致させてください。もちろん保存するため（略

たとえば、

<pre>invoke-virtual {v1, v0, v2}, Lcom/sonyericsson/android/pobox/preferences/POBoxPreference;-&gt;setEnable50Letter(ZZ)V</pre>

これを

<pre>invoke-virtual {v1, v2, v2}, Lcom/sonyericsson/android/pobox/preferences/POBoxPreference;-&gt;setEnable50Letter(ZZ)V</pre>

こう。

追記：タブレットモードがいやだという人向け。オプショナル。

Lcom/sonyericsson/android/pobox/preferences/POBoxPreference;->setEnable12Key(ZZ)V

Lcom/sonyericsson/android/pobox/preferences/POBoxPreference;->setEnableResizable(ZZ)V

上記と同様に引数を一致させます。

&nbsp;

## ・存在しないリソースへの参照を書き換え

該当ファイル、書き換え回数:

  * IWnnImeJaJp.smali ×2
  * SkinData.smali ×3

含まれる文字列で「0x206」を検索します。

IWnnImeJaJp.smaliの場合は、

<pre>invoke-virtual {p0}, Lcom/sonyericsson/android/pobox/core/IWnnImeJaJp;-&gt;getResources()Landroid/content/res/Resources;

    move-result-object v6

    const/high16 v7, 0x206

    invoke-virtual {v6, v7}, Landroid/content/res/Resources;-&gt;getColor(I)I

    move-result v6</pre>

この「invoke-virtual」から「move-result v6」までを次の1行に置き換えます。

<pre>const/16 v6, 0xffffffff</pre>

このとき「move-result v6」の「v6」という引数と、置き換えたあと「const/16 v6, 0xffffffff」の「v6」という引数が一致することに注意してください。これを2ヶ所。

[<img class="alignnone size-full wp-image-133" style="max-width: 100%; height: auto;" alt="Grep and Replace - 書き換え" src="/image/2013/01/2013-01-19-21-35-26_Grep-and-Replace_.png" srcset="/image/2013/01/2013-01-19-21-35-26_Grep-and-Replace_.png 913w, /image/2013/01/2013-01-19-21-35-26_Grep-and-Replace_-300x195.png 300w" sizes="(max-width: 767px) 89vw, (max-width: 1000px) 54vw, (max-width: 1071px) 543px, 580px" />][16]

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

含まれる文字列に「Soma」、置換に「Roboto」と入力します。

「検索」、「置換」、「リストのファイルを一括置換」と進み「確認」「保存確認」に「はい」と答えます。

すると、計16カ所置換されます。

&nbsp;

#  ４．apkのビルド、署名

Grep and Replaceはこの段階になったら終了してください。

## ・apkのビルド

コマンドプロンプトに次のコマンドをコピペし「**Enter**」

<pre>apktool b POBox54 POBox5.4.apk</pre>

##  ・apkの署名

この作業はとても厄介なのでしっかりと確認してください。

コマンドプロンプトに次の行をコピペして「**Enter**」

<pre>keytool -genkey -v -sigalg SHA1withRSA -keyalg RSA -keystore test.keystore -alias testkey -validity 10000</pre>

すると、キーストアのパスワード、姓名、組織名、都市名、州名、国番号を問われるので答えていきます。答えを入力、「**Enter**」という流れです。

パスワード入力中は入力中の文字は表示されません。

必要になった際に使えるようにしておくためキーストアパスワードは忘れないでください。

国番号を答えたあと、 「[いいえ]:」とひょうじされるので「y」と入力し「**Enter**」

その次に再度「**Enter**」と押すと署名に必要なキーができあがります。

&nbsp;

コマンドプロンプトに次の行をコピペして「**Enter**」

<pre>jarsigner -sigalg SHA1withRSA -digestalg SHA1 -keystore test.keystore -verbose POBox5.4.apk testkey</pre>

キーストアのパスワードを聞かれるので入力して「**Enter**」

&nbsp;

次の行も同じです。

<pre>jarsigner -sigalg SHA1withRSA -digestalg SHA1 -keystore test.keystore -verbose POBoxSknSelector.apk testkey</pre>

&nbsp;

## ・apkの最適化(zipalign)

次の2行をコピペして「**Enter**」

<pre>zipalign -v 4 POBox5.4.apk POBox5.4_aligned.apk
zipalign -v 4 POBoxSknSelector.apk POBoxSknSelector_aligned.apk</pre>

&nbsp;

#  ５．インストール

さすがにここは書かなくてもいいかなと思ったのですが一応書いておきます。

ここまででapkの改変そのものは作業が終了しています。

改変済みなのとインストールした方がよいapkは

  * POBox5.4_aligned.apk
  * POBoxSknSelector_aligned.apk
  * POBoxSknMono.apk

&nbsp;

## ・提供元不明のアプリ

Android端末の設定を開きます。

セキュリティをタップし、提供元不明のアプリにチェックを入れます。すると警告が表示されるのでしっかりと読んだ上で「OK」をタップします。

<div>
  <dl id="attachment_48">
    <dt>
      <a href="/image/2012/12/d7abb477ad1bb4688dd2a75134351136.png"><img alt="提供元不明のアプリ" src="/image/2012/12/d7abb477ad1bb4688dd2a75134351136.png" width="360" height="640" /></a>
    </dt>
    
    <dd>
      TabUIなので通常の表示とは違いますがこんな感じです。
    </dd>
  </dl>
</div>

&nbsp;

## ・インストール

adbが使える環境なら話は簡単で、端末をPCにつなぎ次の3行をコマンドプロンプトにコピペ、「**Enter**」で完了です。

<pre>adb install POBoxSknSelector_aligned.apk
adb install POBoxSknMono.apk
adb install POBox5.4_aligned.apk</pre>

問題はそういった環境がない場合です。

たとえばPCと端末をつなぎ、SDカード上に「apk」など適当なフォルダを作り、前述のapkを転送します。

そこに「<a title="ES File Explorer | Google Play" href="https://play.google.com/store/apps/details?id=com.estrongs.android.pop" target="_blank">ES ファイルエクスプローラー</a>」などを使ってアクセスし、インストールしたいapkをタップすればインストール可能です。

&nbsp;

## ・有効化

ほかのIMEを使ったことがある人は知っていると思いますが、POBoxもその例に外れず、インストールしただけでは使うことができません。

そこで何をすればよいかというと、先ほどのようにAndroidの設定を開きます。

言語と入力をタップし、POBox Touch (日本語)にチェックを入れます。ここでも注意が表示されるので納得の上で「OK」をタップしてください。

<div>
  <dl id="attachment_49">
    <dt>
      <a href="/image/2012/12/52c56a315700b05470164497f459245e.png"><img alt="言語と入力" src="/image/2012/12/52c56a315700b05470164497f459245e.png" width="360" height="640" /></a>
    </dt>
    
    <dd>
      IMEが多すぎる件にはあんまり突っ込まないでください…
    </dd>
  </dl>
</div>

&nbsp;

##  ・テスト

無事に起動しましたか？50音キーボードの有効化はできていますか？手書きの軌跡は表示されていますか？

何事もなく成功した人はお疲れ様でした。

失敗してしまった人はトライアンドエラーを繰り返してみてください。それでもだめなら、分かる範囲、できる範囲で対応します。

&nbsp;

##  ・新機能

POBox5.4からタブレット向けのレイアウトとキーボードの表示サイズ調整機能が加わっています。<figure id="attachment_134" style="width: 800px" class="wp-caption alignnone">

[<img class="size-full wp-image-134" style="max-width: 100%; height: auto;" title="POBox5.4 タブレット向けレイアウト" alt="POBox5.4 タブレット向けレイアウト" src="/image/2013/01/Screenshot_2013-01-19-01-31-01-e1358601206165.png" width="800" height="406" srcset="/image/2013/01/Screenshot_2013-01-19-01-31-01-e1358601206165.png 800w, /image/2013/01/Screenshot_2013-01-19-01-31-01-e1358601206165-300x152.png 300w" sizes="(max-width: 767px) 89vw, (max-width: 1000px) 54vw, (max-width: 1071px) 543px, 580px" />][17]<figcaption class="wp-caption-text">POBox5.4 タブレット向けレイアウト</figcaption></figure> <figure id="attachment_135" style="width: 2160px" class="wp-caption alignnone">[<img class=" wp-image-135" style="max-width: 100%; height: auto;" title="POBox5.4 新機能" alt="POBox5.4 新機能" src="/image/2013/01/GN.png" width="2160" height="1280" srcset="/image/2013/01/GN.png 2160w, /image/2013/01/GN-300x177.png 300w, /image/2013/01/GN-1024x606.png 1024w" sizes="(max-width: 767px) 89vw, (max-width: 1000px) 54vw, (max-width: 1071px) 543px, 580px" />][18]<figcaption class="wp-caption-text">POBox5.4 新機能</figcaption></figure> 

&nbsp;

# ６．最後に

ここまで読んでくれてありがとうございました。

初めて書いた記事なので至らない点など多々あると思いますが、なにとぞご容赦のほどを…

[Twitter -- @watiko][19]

&nbsp;

&nbsp;

&nbsp;

<span style="color: #c0c0c0;">一応今回できたもの<a title="POBox5.4_ZD_aligned.rar" href="http://www.mediafire.com/?79woo94ap4n3kvv"><span style="color: #c0c0c0;">POBox5.4_ZD</span></a><br /> タブレットモード無効化版<a title="POBox5.4_notab_ZD.rar" href="http://www.mediafire.com/?6cn66r5bgvv8r1d"><span style="color: #c0c0c0;">POBox5.4_notab_ZD</span></a></span>

<span style="color: #c0c0c0;">ICSかJBのXperia向け(動作未確認・非保証)バックアップの上で実行。<a title="POBox5.4_cwm_signed.zip" href="http://www.mediafire.com/?r27q74c417e7iwa"><span style="color: #c0c0c0;">POBox5.4_cwm_signed.zip</span></a><br /> </span>
  
では。

 [1]: /image/2013/01/Screenshot_2013-01-19-20-36-09.png
 [2]: http://watiko.net/2012/12/28/how-to-make-pobox5-3-work-in-any-devices-without-rooting/ "POBox 5.3をXperia以外の端末でも動くようにする。(root不要) | TypoTypoTypo!"
 [3]: http://www.vector.co.jp/soft/dl/win95/util/se205255.html "ファイルの検索・一括置換 Grep and Replaceのダウンロード : Vector ソフトを探す！"
 [4]: https://code.google.com/p/android-apktool/downloads/list "android-apktool |Google Project Hosting"
 [5]: http://www.oracle.com/technetwork/java/javase/downloads/index.html "JavaSEのダウンロード"
 [6]: http://www.mediafire.com/?vs8sdu35sy7dd46 "APK-modding-tools.zip"
 [7]: /image/2012/12/f79cca1cca4ccd85f013075d15bad303.png
 [8]: /image/2012/12/c895fa206e3fcd705d0b51efa22d0bf5.png
 [9]: /image/2012/12/7a729c2718237dd6b3d82e9a44224ec1.png
 [10]: /image/2013/01/2013-01-19-20-59-09_POBox.png
 [11]: /image/2013/01/2013-01-19-21-15-02_Grep-and-Replace.png
 [12]: /image/2013/01/2013-01-19-21-15-02_Grep-and-Replace2.png
 [13]: /image/2013/01/6e5900aa07e7242384d66b0cd39613d6.png
 [14]: /image/2013/01/64dae904b8be20d3c1ab61ac86b74490.png
 [15]: /image/2012/12/Grep-and-Replace5.png
 [16]: /image/2013/01/2013-01-19-21-35-26_Grep-and-Replace_.png
 [17]: /image/2013/01/Screenshot_2013-01-19-01-31-01-e1358601206165.png
 [18]: /image/2013/01/GN.png
 [19]: https://twitter.com/watiko "@watiko"
