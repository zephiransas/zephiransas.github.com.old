---
layout: post
title: "slickを使う（基礎編）"
date: 2013-04-24 12:42
comments: true
categories: [scala,playframework,slick]
---
最近playframework2.1をちゃんと使いたいなぁと思ったので、Scala含めて色々調べてます。
で、ドキュメントを眺めてて気になったのがO/Rマッパ。play1.2系のころはEBean等のO/Rマッパを利用できたんですが、play2.1系ではなくなってる！一応Anormという仕組みでSQLを発行出来ますが、SQL直書きするのは小学生までよねー？！的な感じ。

で、更に調べてみると、typesafeで[Slick](http://slick.typesafe.com/)というO/Rマッパがあるので、これが良さそうじゃね？ってことで、試してみました。
本来はplayから利用するのがベストですが、実験なのでスタンドアロンで。

## build.sbtとplugin.sbtの準備

まずはsbtで環境を作るところから。以下のようにbuild.sbtを作成
``` scala build.sbt
name := "slicksample"

version := "1.0"

scalaVersion := "2.10.0"

libraryDependencies ++= List(
  "com.typesafe.slick" %% "slick" % "1.0.0",
  "org.slf4j" % "slf4j-nop" % "1.6.4",
  "com.h2database" % "h2" % "1.3.166",
  "com.github.tototoshi" %% "slick-joda-mapper" % "0.1.0"
)
```
11行目では[slick-joda-mapper](http://d.hatena.ne.jp/tototoshi/20130323/1364013170)を指定しています。

Slickでは日付はjava.sql.Dateで扱うらしいのですが、今更java.sql.Dateなんて触りたくないなーと思っていたところ、JodaTimeへマッピングしてくれるプラグインがあったのでこれを使います。

また、今回はIDEにIntelliJを使うので、[sbt-ideaプラグイン](https://github.com/mpeltonen/sbt-idea)を使うように、plugin.sbtを設定します。
``` scala project/plugin.sbt
addSbtPlugin("com.github.mpeltonen" % "sbt-idea" % "1.4.0")
```

あとはsbtからgen-ideaするだけ

``` bash
$ sbt gen-idea
```
ついでにscalaのソースファイル置き場であるsrc/main/scalaディレクトリも作っておきましょう。そして、これをIntelliJに読み込ませます。読み込ませるとこんな感じ。

![image1](/images/20130424/image1.png)

## モデルの作成

次にモデルを作成します。今回はUSERSテーブルにアクセスするUser objectを作成してみます。カラムは

* id - Integer（主キー）
* name - String
* birthday - Option[LocalDate]
ぐらいで。

src/main/scalaにUser.scalaを作成し、以下のように記述します。

``` scala User.scala
import org.joda.time.LocalDate
import scala.slick.driver.H2Driver.simple._
import com.github.tototoshi.slick.JodaSupport._

object User extends Table[(Int, String, Option[LocalDate])]("USERS") {
  def id = column[Int]("id", O.PrimaryKey)
  def name = column[String]("name")
  def birthday = column[Option[LocalDate]]("birthday")
  def * = id ~ name ~ birthday
}
```
まず2行目でSlickのクラスをインポートしておきます。注意すべきは使用するDBに合わせて、インポートするクラスを変えるってところ。今回はDBにH2を使うので
``` scala
import scala.slick.driver.H2Driver.simple._
```
としてます。例えばPostgreSQLを使う場合は
``` scala
import scala.slick.driver.PostgresDriver.simple._
```
とします。

3行目は先述したslick-joda-mapperを使うのに必要です。IntteliJだと、Unusedで警告がでるのがアレですが・・・

## データのINSERTとSELECT
次に、このUserモデルを使ってテーブルを作成しデータをINSERT、その後SELECTして内容を表示するコードを書いてきましょう。
src/main/scalaにMain.scalaを作成し、以下のように記述します。

``` scala src/main/Main.scala
import org.joda.time.LocalDate
import scala.slick.driver.H2Driver.simple._

import Database.threadLocalSession

object Main {
  def main(args:Array[String]):Unit = {
    Database.forURL("jdbc:h2:mem:test1", driver = "org.h2.Driver") withSession {
      User.ddl.create

      User.insert(1, "hoge", Some(new LocalDate(2013, 4, 10)))
      User.insert(2, "fuga", Some(new LocalDate(2013, 4, 11)))

      //select all
      Query(User) foreach { case (id, name, birthday) =>
        println(id + ":" + name + ":" + birthday)
      }

      //name = "hoge"のみ抽出
      val q1 = for{ u <- User if u.name === "hoge" } yield u.name
      q1 foreach println
    }
  }
}
```
まず8行目でDBへの接続を行なっています。

次の9行目ではUSERSテーブルのCREATE文を実行して、テーブルの作成を行なっています。

11,12行目でデータのINSERT。15行目以降で、SELECTを行ない、標準出力で出力しています。

## まとめ

こうして見ると、まだ自分がScalaのListの扱いに慣れてないせいもあって、SELECT周りの書き方にはちょっと違和感を感じますが、O/Rマッパとしての基本的な機能は十分に押さえていそうな感じです。

## 参考サイト
* [Slick1.0.0公式ドキュメント](http://slick.typesafe.com/doc/1.0.0/)
* [tototoshiの日記 - slickガイド](http://d.hatena.ne.jp/tototoshi/20121204/1354615421)