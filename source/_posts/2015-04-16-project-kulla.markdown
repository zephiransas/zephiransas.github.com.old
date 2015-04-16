---
layout: post
title: "Project Kullaを試す"
date: 2015-04-16 12:26:30 +0900
comments: true
categories: Java
---

以前から気になっていたJavaのREPL、Project Kullaを動かしてみました。

REPLとはRead-eval-print loopの略で、CUIからコードを直接入力していって、その場で動作を確認できるツールです。
Rubyであればirbやpryなどが有名ですね。

Project KullaはOpenJDKにて開発されている、JavaのREPL環境をつくるプロジェクトです。
ちなみにこの機能はJDK9で、正式導入される予定になっています。

## JLine2のインストール

早速REPL環境を動かしてみたいところですが、まずは前準備として、Kullaに必要なJLine2というライブラリをビルドします。

ソースコードは[GitHubのリポジトリ](https://github.com/jline/jline2)でホストされていますので

``` bash
git clone git@github.com:jline/jline2.git
cd jline2
mvn install
```

ちなみにJLine2はJDK8以前でないとビルドできないので注意です。

ビルドに成功するとjline2/targetディレクトリにjline-2.13-SNAPSHOT.jarが作成されます。Kullaからは、このjarを利用します。

## JDK9 EAのインストール

KullaのビルドにはJDK9が必要です。[こちら](https://jdk9.java.net/download/)からJDK9をダウンロードし、インストールします。
自分がインストールしたのは、以下のバージョン。

```
java version "1.9.0-ea"
Java(TM) SE Runtime Environment (build 1.9.0-ea-b59)
Java HotSpot(TM) 64-Bit Server VM (build 1.9.0-ea-b59, mixed mode)
```

その後、使用するJAVA_HOMEをJDK9に設定します。

普段、自分はJAVA_HOMEの設定にjava_homeコマンドを使用しているので、.bash_profileに

```
export JAVA_HOME=`/usr/libexec/java_home -v 1.8`
```

としてJDK8を使用しています。今回はJDK9を使いたいので、これを

```
export JAVA_HOME=`/usr/libexec/java_home -v 1.9`
```

とし

``` bash
source ~/.bash_profile
```

として、JDK9を有効にします。

## Kullaのビルド
いよいよKullaのソースをダウンロードしてビルドします。

``` bash
hg clone http://hg.openjdk.java.net/kulla/dev ~/kulla
cd ~/kulla
```

次に、その他必要なソース類を取得します。
``` bash
chmod 755 get_source.sh
./get_source.sh
```

しばらく待つと、終了します。次にビルドスクリプトを環境に合わせて修正します。

``` bash
cd langtools/repl
```

scripts/compileを以下のように修正します。

``` bash scripts/compile
#!/bin/sh
JLINE2LIB=/Users/[ユーザ名]/jline2/target/jline-2.13-SNAPSHOT.jar
JAVAC_BIN_HOME=/Library/Java/JavaVirtualMachines/jdk1.9.0.jdk/Contents/Home/bin

mkdir -p build
$JAVAC_BIN_HOME/javac -Xlint:unchecked -Xdiags:verbose -cp ${JLINE2LIB} -d build src/*/*.java
```

1行目 - OSXの環境に合わせて"#!/bin/sh"に修正
2行目 - jline2のjarを指定
3行目 - JDK9のjavacのあるディレクトリを指定
6行目 - 先頭に"$JAVAC_BIN_HOME"を追加

修正できたら

``` bash
scripts/compile
```

でビルドしましょう。なにもエラーがでなければ、成功しています。

## REPLを実行する

実行前にscripts/runを以下のように修正します。

``` bash scripts/run
#!/bin/sh
JLINE2LIB=/Users/[ユーザ名]/jline2/target/jline-2.13-SNAPSHOT.jar
JAVA_BIN_HOME=/Library/Java/JavaVirtualMachines/jdk1.9.0.jdk/Contents/Home/bin/
$JAVA_BIN_HOME/java -ea -esa -cp build:${JLINE2LIB} tool.Repl "$@"
```

先になおしたスクリプトとほぼ同じです。

修正できたら、早速実行してみましょう。

``` bash
scripts/run
```

すると、以下のようにプロンプトが表示されます。

```
|  Welcome to the Java REPL -- Version 0.411
|  Type /help for help

->
```

あとは普通にJavaのプログラムが書けます！

```
-> System.out.println("Hello!");
```

また、CUIなどと同じようにタブによる補完もできます。

<blockquote class="twitter-tweet" lang="ja"><p><a href="https://twitter.com/zephiransas">@zephiransas</a> ちなみにSHIFT+TABでメソッドのシグネチャが表示されます．&#10;new String([SHIFT+TAB]みたいな感じで</p>&mdash; bitter_fox (@bitter_fox) <a href="https://twitter.com/bitter_fox/status/588512374845411328">2015, 4月 16</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

Shift + Tab補完もなかなかステキ。

当然ですがクラスを定義することもできます。

```
-> class Hoge {
>> public static String fuga(){ return "FUGA!!"; }
>> }
|  Added class Hoge

-> Hoge.fuga();
|  Expression value is: "FUGA!!"
|    assigned to temporary variable $1 of type String
```

面白いのは、**いきなりメソッド定義**もできます！

```
-> int add(int x, int y){ return x + y;}
|  Added method add

-> add(2,3);
|  Expression value is: 5
|    assigned to temporary variable $2 of type int
```

普通に使えそうですね！

またTab補完周りの機能については、我らの[@bitter_foxくん](https://twitter.com/bitter_fox/)が実装に関わってるらしいので、補完機能が怪しかったら、ぜひレポートしてあげてください。

<blockquote class="twitter-tweet" data-conversation="none" lang="ja"><p><a href="https://twitter.com/zephiransas">@zephiransas</a> Tab補完周り僕の少し実装に関わってるんで，良ければフィードバックください！</p>&mdash; bitter_fox (@bitter_fox) <a href="https://twitter.com/bitter_fox/status/588512213050101760">2015, 4月 16</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

## 参考にしたリンク
- Java 9 REPL – Getting started guide - http://www.jclarity.com/2015/04/15/java-9-repl-getting-started-guide/
- REPLで遊ぼう - http://d.hatena.ne.jp/bitter_fox/20150331/1427754868
