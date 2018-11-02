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
[markdown]
  
ところどころ手を加える必要があったので備忘録をかねて。(Cygwinと書きましたがほかの環境でも参考になると思います。)

Cygwin (またはGNU/LinuxなどのNDKを動かせるもの) の導入
  
&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8211;

今回は省きます。と、言ってもほとんど初期状態でも大丈夫な気がしますが。
  
ただ、\`Cygwin\`や\`MSYS/MinGW\`などのWindows上の環境でやるには管理者権限が必要です。
  
ビルドする際にシンボリックリンクを張りながら進むものがあってそれでとまります。NDKのツールが解釈できない模様。

&#8220;\`sh
  
echo &#8220;&#8221; >> ~/.zshrc
  
echo &#8220;export CYGWIN=&#8217;dosfilewarning winsymlinks:native'&#8221; >> ~/.zshrc
  
&#8220;\`

などとしておくと好いでしょう。ただ、管理者権限で起動するとなぜか\`dosfilewarning\`が効かないのつらい。改めて\`source ~/.zshrc\`とか\`export CYGWIN=以下略\`とかすると大丈夫なんだけどね…

<!--more-->

Android NDKの導入
  
&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;-

\### NDKの配置
  
* \[Android NDK | Android Developer\](http://developer.android.com/tools/sdk/ndk/index.html &#8220;Android NDK | Android Developer&#8221;) 

自分の環境にあったNDKをダウンロードして、好きな場所に伸張します。
  
ここでは\`android-ndk-r9d-windows-x86_64.zip\`を\`~/Dev/Android/\`で伸張します。

&#8220;\`sh
  
cd ~/Dev/Android/
  
unzip android-ndk-r9d-windows-x86_64
  
&#8220;\`

\### 環境変数などの設定
  
NDKのパスは好きにしてもらってかまわないのですが(先程の例だと\`NDK_ROOT=$HOME/Dev/Android/andrioid-ndk-r9d\`になります)、irbがRubyをビルドしたときのprefixを見に行くのでANDBOXはそのときに置くパスの一部にすると、後々楽に作業ができます。
  
(irbなどのshebangが決め打ちされます、わざわざ書き換えるのは面倒。それこそスクリプト書けばいいのだけれど)
  
たとえば、\[id:rattcv\](http://d.hatena.ne.jp/rattcv &#8220;rattcvの日記。&#8221;)さんのrboxに組み込むなら、\`ANDBOX=/data/data/jackpal.androidterm/shared_prefs/rbox\`などとすると好いでしょう。(Androidのバージョンによっては違うかもしれないので各自確認のこと)

ちなみにplatformsは\[このページ\](http://developer.android.com/guide/topics/manifest/uses-sdk-element.html &#8220;<uses-sdk\> | Android Developers&#8221;)のAPI Levelに対応しています。

&#8220;\`sh
  
echo &#8220;&#8221; >> ~/.zshrc
  
echo &#8220;export ANDBOX=/path/to/your/directoory&#8221; >> ~/.zshrc
  
echo &#8220;export NDK_ROOT=/path/to/your/ndk&#8221; >> ~/.zshrc
  
echo &#8220;export SYSROOT=$NDK_ROOT/platforms/android-9&#8221; >> ~/.zshrc
  
source ~/.zshrc
  
&#8220;\`

\### スタンドアローンのビルドツールをインストール
  
動かすAndroidのアーキテクスチャはarmということで進めていきます。

&#8220;\`sh
  
$NDK_ROOT/build/tools/make-standalone-toolchain.sh \
    
&#8211;platform=android-9 \
    
&#8211;arch=arm \
    
&#8211;install-dir=/usr/local/android\_ndk\_standalone-r9d \
    
&#8211;ndk-dir=$NDK_ROOT

echo &#8220;export PATH=$PATH:/usr/local/android\_ndk\_standalone-r9d/bin&#8221; >> ~/.zshrc
  
&#8220;\`

事前に必要なもののビルド
  
&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;

と書きましたが、実際にはなくてもRubyのビルドはできます。ただ、一部機能が使えなくなるなど制約が生まれるので一緒にビルドしておくことをお勧めします。

|対象|概要|
  
|:&#8212;|:&#8211;:|
  
|libffi|Fiddleを使うため。|
  
|ncurses|Ruby2.1.0からはgem(curses)になった。|
  
|openssl|gemとか使うのに必要なので。|
  
|readline|irbとかpryとかで直前の入力を再利用したい。|

pryで問題が出るので私はlessもビルドしました。
  
(ビルドオプションは最大公約数的にするといいかも。あと-Iとか-Lを忘れずに。)

\### 準備
  
ビルドするフォルダの作成

&#8220;\`sh
  
mkdir -p ~/tmp/android/
  
cd ~/tmp/android/
  
&#8220;\`

\### libffi
  
特になし

&#8220;\`sh
  
cd ~/tmp/android/

wget ftp://sourceware.org/pub/libffi/libffi-3.0.13.tar.gz
  
tar xfv libffi-3.0.13.tar.gz
  
cd libffi-3.0.13

export CC=arm-linux-androideabi-gcc
  
export CXX=arm-linux-androideabi-g++
  
export CFLAGS=&#8217;-march=armv5te -msoft-float -Os -static-libgcc&#8217;
  
export LDFLAGS=&#8217;-static-libgcc&#8217;

CC=$CC ./configure \
    
&#8211;host=arm-linux-androideabi \
    
&#8211;with-sysroot=$SYSROOT \
    
&#8212;shared \
    
&#8211;enable-static \
    
&#8211;prefix=$ANDBOX/usr/local/

make && make install
  
&#8220;\`

\### ncurses
  
特になし

&#8220;\`sh
  
cd ~/tmp/android/

wget http://ftp.gnu.org/pub/gnu/ncurses/ncurses-5.9.tar.gz
  
tar xfv ncurses-5.9.tar.gz
  
cd ncurses-5.9

export CC=arm-linux-androideabi-gcc
  
export CXX=arm-linux-androideabi-g++
  
export CFLAGS=&#8217;-march=armv5te -msoft-float -Os -static-libgcc&#8217;
  
export LDFLAGS=&#8217;-static-libgcc&#8217;

ac\_cv\_header\_locale\_h=no ./configure \
    
&#8211;host=arm-linux-androideabi \
    
&#8211;without-cxx-binding \
    
&#8211;enable-multibyte \
    
&#8211;enable-static \
    
&#8212;shared \
    
&#8211;prefix=$ANDBOX/usr/local/

make && make install
  
&#8220;\`

\### openssl
  
CygwinだとMakefileに手直しが必要。
  
no-sharedにすると後々問題が発生。
  
(最近のAndroid=>4.3だとopenssl-1.x.xのほうがいいのかもしれない。\[参考\](http://d.hatena.ne.jp/android-memo/20130817 &#8220;OpenSSL 0.9.8yをAndroid NDKでクロスコンパイルする&#8221;))

&#8220;\`sh
  
cd ~/tmp/android/

wget https://www.openssl.org/source/openssl-0.9.8y.tar.gz
  
tar xfv openssl-0.9.8y.tar.gz
  
cd openssl-0.9.8y

export CC=gcc
  
export CXX=g++
  
export CFLAGS=&#8217;-march=armv5te -msoft-float -Os -static-libgcc&#8217;
  
export LDFLAGS=&#8217;-static-libgcc&#8217;

CROSS_COMPILE=arm-linux-androideabi- ./Configure \
    
&#8211;prefix=$ANDBOX/usr/local/ threads zlib shared no-asm linux-generic32
  
&#8220;\`

&#8220;\`diff
  
\# ここまででMakefileができるので少々手直し
  
\# ドキュメント何ぞ不要
  
-install: all install\_docs install\_sw
  
+install: all install_sw

\# CygwinでNDKを使っているとパスの解釈が…
  
-$(RANLIB) $(INSTALL_PREFIX)$(INSTALLTOP)/$(LIBDIR)/$$i.new; \
  
+$(RANLIB) \`cygpath -m $(INSTALL_PREFIX)$(INSTALLTOP)/$(LIBDIR)/$$i.new\`; \
  
&#8220;\`

&#8220;\`sh
  
make && make install
  
&#8220;\`

\### readline
  
これもCygwinだとMakefileに手直しが必要。
  
\`./support/config.sub\`が古くAndroid用の設定がないので対策。
  
ちょっとパッチもあてる。 

&#8220;\`sh
  
mkdir ~/tmp/android/readline
  
cd ~/tmp/android/readline

wget ftp://ftp.cwru.edu/pub/bash/readline-6.3.tar.gz
  
tar xfv readline-6.3.tar.gz
  
cd readline-6.3

wget -O support/config.guess &#8216;http://git.savannah.gnu.org/gitweb/?p=config.git;a=blob_plain;f=config.guess;hb=HEAD&#8217;
  
wget -O support/config.sub &#8216;http://git.savannah.gnu.org/gitweb/?p=config.git;a=blob_plain;f=config.sub;hb=HEAD&#8217;
  
&#8220;\`

パッチ:\[参考\](http://permalink.gmane.org/gmane.comp.gnu.readline/478)

&#8220;\`diff
  
&#8212; a/aclocal.m4
  
+++ b/aclocal.m4
  
@@ -1784,7 +1784,7 @@
          
exit (w == 0); /\* exit 0 if wcwidth broken \*/
  
}
  
],
  
-bash\_cv\_wcwidth\_broken=yes, bash\_cv\_wcwdith\_broken=no,bash\_cv\_wcwidth_broken=no)])
  
+bash\_cv\_wcwidth\_broken=yes, bash\_cv\_wcwidth\_broken=no,bash\_cv\_wcwidth_broken=no)])
   
if test &#8220;$bash\_cv\_wcwidth_broken&#8221; = yes; then
           
AC\_DEFINE(WCWIDTH\_BROKEN, 1, [wcwidth is usually not broken])
   
fi
  
&#8220;\`

&#8220;\`sh
  
autoconf

export CC=arm-linux-androideabi-gcc
  
export CXX=arm-linux-androideabi-g++
  
export CFLAGS=&#8221;-march=armv7-a -msoft-float -Os -static-libgcc -I$ANDBOX/usr/local/include&#8221;
  
export LDFLAGS=&#8221;-static-libgcc -L$ANDBOX/usr/local/lib -lncurses&#8221;
  
export SYSROOT=$NDK_ROOT/platforms/android-9

./configure \
    
&#8211;host=arm-linux-androideabi \
    
&#8211;with-curses \
    
&#8211;with-sysroot=$SYSROOT \
    
&#8211;enable-multibyte \
    
&#8211;enable-static \
    
&#8211;enable-shared \
    
&#8211;prefix=$ANDBOX/usr/local/stow/readline-6.3
  
&#8220;\`

&#8220;\`diff
  
\# MakefileをCygwin(ry
  
&#8211; -test -n &#8220;$(RANLIB)&#8221; && $(RANLIB) \`$@
  
+ -test -n &#8220;$(RANLIB)&#8221; && $(RANLIB) \`cygpath -m $@\`
  
&#8211; -test -n &#8220;$(RANLIB)&#8221; && $(RANLIB) \`$@
  
+ -test -n &#8220;$(RANLIB)&#8221; && $(RANLIB) \`cygpath -m $@\`
  
&#8211; -test -n &#8220;$(RANLIB)&#8221; && $(RANLIB) $(DESTDIR)$(libdir)/libreadline.a
  
+ -test -n &#8220;$(RANLIB)&#8221; && $(RANLIB) \`cygpath -m $(DESTDIR)$(libdir)/libreadline.a\`
  
&#8211; -test -n &#8220;$(RANLIB)&#8221; && $(RANLIB) $(DESTDIR)$(libdir)/libhistory.a
  
+ -test -n &#8220;$(RANLIB)&#8221; && $(RANLIB) \`cygpath -m $(DESTDIR)$(libdir)/libhistory.a\`
  
&#8220;\`

&#8220;\`sh
  
make && make install
  
&#8220;\`

Ruby
  
&#8212;&#8212;

\### ビルド
  
configureをちょっと書き換えれば(たぶん)大丈夫。
  
ただ、Cygwinだとパスの解釈がだめだったのでconfigure実行時の記述はCygwin用に調整したもの、注意。(見たら分かると思うけど)
  
具体的には&#8220;&#8211;with-opt-dir=\`cygpath -m $ANDBOX/local\` &#8220;

&#8220;\`sh
  
cd ~/tmp/android/

wget http://cache.ruby-lang.org/pub/ruby/2.1/ruby-2.1.0.tar.gz
  
tar xfv ruby-2.1.0.tar.gz
  
cd ruby-2.1.0
  
&#8220;\`

* \[RubyをAndroid用にビルドする | KMC Staff Blog\](http://blog.kmckk.com/archives/3867590.html &#8220;RubyをAndroid用にビルドする&#8221;) 

このページのconfigureに対するパッチを参考にすると。

&#8220;\`diff
  
&#8212; ruby-2.1.0/configure 2013-12-26 00:25:39.000000000 +0900
  
+++ ruby-2.1.0_mod/configure 2014-02-17 15:13:16.758781200 +0900
  
@@ -18846,7 +18846,9 @@
     
$as\_echo\_n &#8220;(cached) &#8221; >&6
   
else
     
if test &#8220;$cross_compiling&#8221; = yes; then :
  
&#8211; as\_fn\_error $? &#8220;cannot check setpgrp when cross compiling&#8221; &#8220;$LINENO&#8221; 5
  
+ #as\_fn\_error $? &#8220;cannot check setpgrp when cross compiling&#8221; &#8220;$LINENO&#8221; 5
  
+ # hack for Android
  
+ ac\_cv\_func\_setpgrp\_void=yes
   
else
     
cat confdefs.h &#8211; <<\_ACEOF >conftest.$ac\_ext
   
/\* end confdefs.h. \*/
  
@@ -19928,7 +19930,11 @@
     
nacl) :
        
LDSHARED=&#8217;$(CC) -shared&#8217; ;; #(
     
*) :
  
&#8211; : ${LDSHARED=&#8217;$(LD)&#8217;} ;;
  
+# : ${LDSHARED=&#8217;$(LD)&#8217;} ;;
  
+# : ${LDSHARED=&#8217;ld&#8217;} ;;
  
+# hack for Android
  
+ : ${LDSHARED=&#8217;$(CC) -shared&#8217;} ;;
  
+
   
esac
     
{ $as\_echo &#8220;$as\_me:${as\_lineno-$LINENO}: result: $rb\_cv_dlopen&#8221; >&5
   
$as\_echo &#8220;$rb\_cv_dlopen&#8221; >&6; }
  
&#8220;\`

こんな感じになった。参考ページでは他の箇所もいじっていたけど、とりあえずこれでビルドできる。

&#8220;\`sh
  
mkdir ~/tmp/android/ruby-build/
  
cd ~/tmp/android/ruby-build/

export CC=arm-linux-androideabi-gcc
  
export CXX=arm-linux-androideabi-g++
  
export CFLAGS=&#8217;-march=armv5te -msoft-float -DYAML\_DECLARE\_STATIC&#8217;
  
export LDFLAGS=&#8217;-static-libgcc&#8217;
  
export LDSHARED=&#8217;arm-linux-androideabi-gcc -shared -static-libgcc&#8217;

../ruby-2.1.0/configure \
    
&#8211;host=arm-linux-androideabi \
    
&#8211;enable-shared \
    
&#8212;ipv6 \
    
&#8212;install-doc \
    
&#8211;with-sysroot=$SYSROOT \
    
&#8211;with-opt-dir=\`cygpath -m $ANDBOX/usr/local\` \
    
&#8211;prefix=$ANDBOX/usr/local

make && make install
  
&#8220;\`

そして実機へ…
  
&#8212;&#8212;&#8212;&#8212;&#8212;

&#8220;\`sh
  
cd $ANDBOX/usr/local/
  
tar czfv ruby-2.1.0\_for\_android.tar.gz bin/\* lib/\*
  
&#8220;\`

これであとは\`ruby-2.1.0\_for\_android.tar.gz\`をAndroidの好きな場所に展開してパスとか通すだけですね！
  
ちなみに最初のほうで言っていたrboxの場合は、SDカードのルートに圧縮ファイルを置いたとするとrboxで

&#8220;\`sh
  
pwd #=> /data/data/jackpal.androidterm/shared_prefs/rbox/home つまり $ANDBOX/home なんです
  
mkdir ../usr/local
  
cd ../usr/local
  
cp /sdcard/ruby-2.1.0\_for\_android.tar.gz ./
  
tar xfv -2.1.0\_for\_android.tar.gz

echo &#8220;&#8221; >> ~/.zshrc
  
echo &#8220;export RBOX=/data/data/jackpal.androidterm/shared_prefs&#8221; >> ~/.zshrc
  
echo &#8220;export PATH=$RBOX/usr/local/bin:$PATH&#8221; >> ~/.zshrc
  
echo &#8220;export LD\_LIBRARY\_PATH=$LD\_LIBRARY\_PATH:$RBOX/usr/local/lib&#8221; >> ~/.zshrc

touch ~/.gemrc
  
echo &#8220;gem: &#8211;no-rdoc &#8211;no-ri&#8221; >> ~/.gemrc
  
&#8220;\`

これでAndroid上のRubyで便利な有理数リテラルや、キーワード引数などの新機能。
  
irbやpryで履歴が参照できるすばらしさを味わうことができるでしょう。 

&#8212;

P.S.:
  
記憶を辿りながら書いているので間違いや、あるいはtypoがあるかもしれません。
  
もし見つけたら教えていただけると幸いです。

\[@watiko\](https://twitter.com/watiko)

[/markdown]