---
title: CygwinでRuby 2.1.0をクロスコンパイル with Android NDK
author: watiko
type: post
date: 2014-02-22T07:36:56+00:00
url: /2014/02/22/crosscompile-ruby-2-1-0-for-android-with-ndk-on-cygwin/
pdrp_attributionLocation:
  - end
categories:
  - Android
  - Application
  - Tool

---

ところどころ手を加える必要があったので備忘録をかねて。(Cygwinと書きましたがほかの環境でも参考になると思います。)

Cygwin (またはGNU/LinuxなどのNDKを動かせるもの) の導入
--------------------------------------------------------

今回は省きます。と、言ってもほとんど初期状態でも大丈夫な気がしますが。  
ただ、`Cygwin`や`MSYS/MinGW`などのWindows上の環境でやるには管理者権限が必要です。  
ビルドする際にシンボリックリンクを張りながら進むものがあってそれでとまります。NDKのツールが解釈できない模様。

```sh
echo "" >> ~/.zshrc
echo "export CYGWIN='dosfilewarning winsymlinks:native'" >> ~/.zshrc
```

などとしておくと好いでしょう。ただ、管理者権限で起動するとなぜか`dosfilewarning`が効かないのつらい。改めて`source ~/.zshrc`とか`export CYGWIN=以下略`とかすると大丈夫なんだけどね…

<!--more-->

Android NDKの導入
-------------------

### NDKの配置
* [Android NDK | Android Developer](http://developer.android.com/tools/sdk/ndk/index.html "Android NDK | Android Developer")  

自分の環境にあったNDKをダウンロードして、好きな場所に伸張します。
ここでは`android-ndk-r9d-windows-x86_64.zip`を`~/Dev/Android/`で伸張します。

```sh
cd ~/Dev/Android/
unzip android-ndk-r9d-windows-x86_64
```

### 環境変数などの設定
NDKのパスは好きにしてもらってかまわないのですが(先程の例だと`NDK_ROOT=$HOME/Dev/Android/andrioid-ndk-r9d`になります)、irbがRubyをビルドしたときのprefixを見に行くのでANDBOXはそのときに置くパスの一部にすると、後々楽に作業ができます。  
(irbなどのshebangが決め打ちされます、わざわざ書き換えるのは面倒。それこそスクリプト書けばいいのだけれど)  
たとえば、[id:rattcv](http://d.hatena.ne.jp/rattcv "rattcvの日記。")さんのrboxに組み込むなら、`ANDBOX=/data/data/jackpal.androidterm/shared_prefs/rbox`などとすると好いでしょう。(Androidのバージョンによっては違うかもしれないので各自確認のこと)

ちなみにplatformsは[このページ](http://developer.android.com/guide/topics/manifest/uses-sdk-element.html "<uses-sdk\> | Android Developers")のAPI Levelに対応しています。

```sh
echo "" >> ~/.zshrc
echo "export ANDBOX=/path/to/your/directoory" >> ~/.zshrc
echo "export NDK_ROOT=/path/to/your/ndk" >> ~/.zshrc
echo "export SYSROOT=$NDK_ROOT/platforms/android-9" >> ~/.zshrc
source ~/.zshrc
```

### スタンドアローンのビルドツールをインストール
動かすAndroidのアーキテクスチャはarmということで進めていきます。

```sh
$NDK_ROOT/build/tools/make-standalone-toolchain.sh \
  --platform=android-9 \
  --arch=arm \
  --install-dir=/usr/local/android_ndk_standalone-r9d \
  --ndk-dir=$NDK_ROOT

echo "export PATH=$PATH:/usr/local/android_ndk_standalone-r9d/bin" >> ~/.zshrc
```

 事前に必要なもののビルド
---------------------------

と書きましたが、実際にはなくてもRubyのビルドはできます。ただ、一部機能が使えなくなるなど制約が生まれるので一緒にビルドしておくことをお勧めします。

|対象|概要|
|:---|:--:|
|libffi|Fiddleを使うため。|
|ncurses|Ruby2.1.0からはgem(curses)になった。|
|openssl|gemとか使うのに必要なので。|
|readline|irbとかpryとかで直前の入力を再利用したい。|

pryで問題が出るので私はlessもビルドしました。  
(ビルドオプションは最大公約数的にするといいかも。あと-Iとか-Lを忘れずに。)

### 準備
ビルドするフォルダの作成

```sh
mkdir -p ~/tmp/android/
cd ~/tmp/android/
```

### libffi
特になし

```sh
cd ~/tmp/android/

wget ftp://sourceware.org/pub/libffi/libffi-3.0.13.tar.gz
tar xfv libffi-3.0.13.tar.gz
cd libffi-3.0.13

export CC=arm-linux-androideabi-gcc
export CXX=arm-linux-androideabi-g++
export CFLAGS='-march=armv5te -msoft-float -Os -static-libgcc'
export LDFLAGS='-static-libgcc'

CC=$CC ./configure \
  --host=arm-linux-androideabi \
  --with-sysroot=$SYSROOT \
  ---shared \
  --enable-static \
  --prefix=$ANDBOX/usr/local/

make && make install
```

### ncurses
特になし

```sh
cd ~/tmp/android/

wget http://ftp.gnu.org/pub/gnu/ncurses/ncurses-5.9.tar.gz
tar xfv ncurses-5.9.tar.gz
cd ncurses-5.9

export CC=arm-linux-androideabi-gcc
export CXX=arm-linux-androideabi-g++
export CFLAGS='-march=armv5te -msoft-float -Os -static-libgcc'
export LDFLAGS='-static-libgcc'

ac_cv_header_locale_h=no ./configure \
  --host=arm-linux-androideabi \
  --without-cxx-binding \
  --enable-multibyte \
  --enable-static \
  ---shared \
  --prefix=$ANDBOX/usr/local/

make && make install
```

### openssl
CygwinだとMakefileに手直しが必要。  
no-sharedにすると後々問題が発生。  
(最近のAndroid=>4.3だとopenssl-1.x.xのほうがいいのかもしれない。[参考](http://d.hatena.ne.jp/android-memo/20130817 "OpenSSL 0.9.8yをAndroid NDKでクロスコンパイルする"))

```sh
cd ~/tmp/android/

wget https://www.openssl.org/source/openssl-0.9.8y.tar.gz
tar xfv openssl-0.9.8y.tar.gz
cd openssl-0.9.8y

export CC=gcc
export CXX=g++
export CFLAGS='-march=armv5te -msoft-float -Os -static-libgcc'
export LDFLAGS='-static-libgcc'

CROSS_COMPILE=arm-linux-androideabi- ./Configure \
  --prefix=$ANDBOX/usr/local/ threads zlib shared no-asm linux-generic32
```

```diff
# ここまででMakefileができるので少々手直し
# ドキュメント何ぞ不要
-install: all install_docs install_sw
+install: all install_sw

# CygwinでNDKを使っているとパスの解釈が…
-$(RANLIB) $(INSTALL_PREFIX)$(INSTALLTOP)/$(LIBDIR)/$$i.new; \
+$(RANLIB) `cygpath -m $(INSTALL_PREFIX)$(INSTALLTOP)/$(LIBDIR)/$$i.new`; \
```

```sh
make && make install
```

### readline
これもCygwinだとMakefileに手直しが必要。
`./support/config.sub`が古くAndroid用の設定がないので対策。  
ちょっとパッチもあてる。  

```sh
mkdir ~/tmp/android/readline
cd ~/tmp/android/readline

wget ftp://ftp.cwru.edu/pub/bash/readline-6.3.tar.gz
tar xfv readline-6.3.tar.gz
cd readline-6.3

wget -O support/config.guess 'http://git.savannah.gnu.org/gitweb/?p=config.git;a=blob_plain;f=config.guess;hb=HEAD'
wget -O support/config.sub 'http://git.savannah.gnu.org/gitweb/?p=config.git;a=blob_plain;f=config.sub;hb=HEAD'
```

パッチ:[参考](http://permalink.gmane.org/gmane.comp.gnu.readline/478)

```diff
--- a/aclocal.m4
+++ b/aclocal.m4
@@  -1784,7 +1784,7  @@
        exit (w == 0);  /* exit 0 if wcwidth broken */
}
],
-bash_cv_wcwidth_broken=yes, bash_cv_wcwdith_broken=no,bash_cv_wcwidth_broken=no)])
+bash_cv_wcwidth_broken=yes, bash_cv_wcwidth_broken=no,bash_cv_wcwidth_broken=no)])
 if test "$bash_cv_wcwidth_broken" = yes; then
         AC_DEFINE(WCWIDTH_BROKEN, 1, [wcwidth is usually not broken])
 fi
```

```sh
autoconf

export CC=arm-linux-androideabi-gcc
export CXX=arm-linux-androideabi-g++
export CFLAGS="-march=armv7-a -msoft-float -Os -static-libgcc -I$ANDBOX/usr/local/include"
export LDFLAGS="-static-libgcc -L$ANDBOX/usr/local/lib -lncurses"
export SYSROOT=$NDK_ROOT/platforms/android-9

./configure \
  --host=arm-linux-androideabi \
  --with-curses \
  --with-sysroot=$SYSROOT \
  --enable-multibyte \
  --enable-static \
  --enable-shared \
  --prefix=$ANDBOX/usr/local/stow/readline-6.3
```

```diff
# MakefileをCygwin(ry
-  -test -n "$(RANLIB)" && $(RANLIB) `$@
+  -test -n "$(RANLIB)" && $(RANLIB) `cygpath -m $@`
-  -test -n "$(RANLIB)" && $(RANLIB) `$@
+  -test -n "$(RANLIB)" && $(RANLIB) `cygpath -m $@`
-  -test -n "$(RANLIB)" && $(RANLIB) $(DESTDIR)$(libdir)/libreadline.a
+  -test -n "$(RANLIB)" && $(RANLIB) `cygpath -m $(DESTDIR)$(libdir)/libreadline.a`
-  -test -n "$(RANLIB)" && $(RANLIB) $(DESTDIR)$(libdir)/libhistory.a 
+  -test -n "$(RANLIB)" && $(RANLIB) `cygpath -m $(DESTDIR)$(libdir)/libhistory.a`
```

```sh
make && make install
```

Ruby
------

### ビルド
configureをちょっと書き換えれば(たぶん)大丈夫。  
ただ、Cygwinだとパスの解釈がだめだったのでconfigure実行時の記述はCygwin用に調整したもの、注意。(見たら分かると思うけど)  
具体的には``--with-opt-dir=`cygpath -m $ANDBOX/local` ``

```sh
cd ~/tmp/android/

wget http://cache.ruby-lang.org/pub/ruby/2.1/ruby-2.1.0.tar.gz
tar xfv ruby-2.1.0.tar.gz
cd ruby-2.1.0
```

* [RubyをAndroid用にビルドする | KMC Staff Blog](http://blog.kmckk.com/archives/3867590.html "RubyをAndroid用にビルドする")  

このページのconfigureに対するパッチを参考にすると。

```diff
--- ruby-2.1.0/configure     2013-12-26 00:25:39.000000000 +0900
+++ ruby-2.1.0_mod/configure        2014-02-17 15:13:16.758781200 +0900
@@ -18846,7 +18846,9 @@
   $as_echo_n "(cached) " >&6
 else
   if test "$cross_compiling" = yes; then :
-  as_fn_error $? "cannot check setpgrp when cross compiling" "$LINENO" 5
+  #as_fn_error $? "cannot check setpgrp when cross compiling" "$LINENO" 5
+  # hack for Android
+  ac_cv_func_setpgrp_void=yes
 else
   cat confdefs.h - <<_ACEOF >conftest.$ac_ext
 /* end confdefs.h.  */
@@ -19928,7 +19930,11 @@
   nacl) :
      LDSHARED='$(CC) -shared'  ;; #(
   *) :
-       : ${LDSHARED='$(LD)'} ;;
+#      : ${LDSHARED='$(LD)'} ;;
+#      : ${LDSHARED='ld'} ;;
+# hack for Android
+       : ${LDSHARED='$(CC) -shared'} ;;
+
 esac
   { $as_echo "$as_me:${as_lineno-$LINENO}: result: $rb_cv_dlopen" >&5
 $as_echo "$rb_cv_dlopen" >&6; }
```

こんな感じになった。参考ページでは他の箇所もいじっていたけど、とりあえずこれでビルドできる。

```sh
mkdir ~/tmp/android/ruby-build/
cd ~/tmp/android/ruby-build/

export CC=arm-linux-androideabi-gcc
export CXX=arm-linux-androideabi-g++
export CFLAGS='-march=armv5te -msoft-float -DYAML_DECLARE_STATIC'
export LDFLAGS='-static-libgcc'
export LDSHARED='arm-linux-androideabi-gcc -shared -static-libgcc'

../ruby-2.1.0/configure \
  --host=arm-linux-androideabi \
  --enable-shared \
  ---ipv6 \
  ---install-doc \
  --with-sysroot=$SYSROOT \
  --with-opt-dir=`cygpath -m $ANDBOX/usr/local` \
  --prefix=$ANDBOX/usr/local

make && make install
```

そして実機へ…
---------------

```sh
cd $ANDBOX/usr/local/
tar czfv ruby-2.1.0_for_android.tar.gz bin/* lib/*
```

これであとは`ruby-2.1.0_for_android.tar.gz`をAndroidの好きな場所に展開してパスとか通すだけですね！  
ちなみに最初のほうで言っていたrboxの場合は、SDカードのルートに圧縮ファイルを置いたとするとrboxで

```sh
pwd #=> /data/data/jackpal.androidterm/shared_prefs/rbox/home つまり $ANDBOX/home なんです
mkdir ../usr/local
cd ../usr/local
cp /sdcard/ruby-2.1.0_for_android.tar.gz ./
tar xfv -2.1.0_for_android.tar.gz

echo "" >> ~/.zshrc
echo "export RBOX=/data/data/jackpal.androidterm/shared_prefs" >> ~/.zshrc
echo "export PATH=$RBOX/usr/local/bin:$PATH" >> ~/.zshrc
echo "export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$RBOX/usr/local/lib" >> ~/.zshrc

touch ~/.gemrc
echo "gem: --no-rdoc --no-ri" >> ~/.gemrc
```

これでAndroid上のRubyで便利な有理数リテラルや、キーワード引数などの新機能。  
irbやpryで履歴が参照できるすばらしさを味わうことができるでしょう。  

---

P.S.:  
記憶を辿りながら書いているので間違いや、あるいはtypoがあるかもしれません。  
もし見つけたら教えていただけると幸いです。

[@watiko](https://twitter.com/watiko)
