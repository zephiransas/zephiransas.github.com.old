---
layout: post
title: "lambda-behaveでテストを書こう"
date: 2014-12-16 18:17:15 +0900
comments: true
categories: Java
---

これは[Javaアドベントカレンダー2014](http://qiita.com/advent-calendar/2014/java)の12/16分の記事です。

昨日は[grimrose](https://github.com/grimrose)さんの、[[書評] Gradle徹底入門](http://grimrose.blogspot.jp/2014/12/gradle.html) でした。

明日は[@com4dc](https://twitter.com/com4dc)さんの、[はじめて触るStreamの世界](http://dev.classmethod.jp/server-side/what-a-wonderful-stream-world/) です。

自分はJavaのテストフレームワークである、[lambda-behave](https://github.com/RichardWarburton/lambda-behave)について紹介します。

自分は普段はRailsでの開発を行っているのですが、現場では主にRSpecを使ってテストを記述しています。RSpecでのテストは以下のような感じです。

``` ruby
describe 'Sample' do
  context 'hogeメソッドについて' do
    it 'fugaを返すこと' do
      Sample.hoge.should == "fuga"
    end
  end
end
```

RSpecでは上記のようにDSLを使って、なにをテストしているかを構造的に記述することができます。
lambda-behaveを使うと、このようなDSLっぽい記述のテストを、Java8のLambdaを使って書くことができるようになります。

## 最初のテスト

まずはテスト対象となるメソッドを準備します。

``` java
public class Sample {
    public static int includeTax(Integer price) {
        return 0;
    }
}
```

上記のようなstaticなメソッドを準備します。includeTaxメソッドは引数を一つ取り、その税込み金額を返すメソッドとします。**実にギョーミーですね！**

今回はTDD的なノリで実装していきますので、ここでは中身の実装はおこないません。


それでは実際のテストを書いて行きましょう。ここでのテストシナリオは

- includeTaxメソッドに100を渡した場合に、108が返ってくること

をテストするとします。これをlambda-behaveで書くと、以下のようになります。

{% raw %}
``` java
import static com.insightfullogic.lambdabehave.Suite.*;

@RunWith(JunitSuiteRunner.class)
public class SampleSpec {{ 
    describe("includeTax", it -> {
        it.should("税込み価格が取得できること", expect ->
            expect.that(Sample.includeTax(100)).is(108)
        );
    });
}}
```
{% endraw %}

static importを使ってlambda-behaveのメソッドを使えるようにし、こられを使って記述していきます。

JUnitのランナーも用意されていますので、これを@RunWithで指定すれば、IDEからも簡単にテストを実行可能です。早速、テストを実行してみましょう。

![screen1](/images/20141216/screen1.png)

includeTaxは、まだ実装をおこなっていませんので、当然このとおりテストが失敗します。

次に、includeTaxを実装してみます。

``` java
public class Sample {
    public static int includeTax(Integer price) {
        final float TAX_RATE = 0.08;
        return (int)Math.floor(price * (1 + TAX_RATE));
    }
}
```

再度、テストを実行すれば、テストが成功しています。

![screen2](/images/20141216/screen2.png)

## 複数のテストデータでチェック

先の例では1つの値でしかテストしませんでしたが、lambda-behaveでは同時に複数の値でテストすることもできます。以下のようになります。

``` java
it.uses(100, 108)
  .and(200, 216)
  .toShow("税込み価格が取得できること", (expect, price, includeTax) -> {
     expect.that(Sample.includeTax(price)).is(includeTax);
  });
```

itのあとにuseとandをチェインして値を準備し、これを使ってtoShowメソッド内でテストを行います。toShowメソッド内ではlambdaの引数で準備した値を利用できますので、lambda内でその値をexpectするようにしています。

## 生成した値でチェック

lambda-behaveではランダムな値を生成する機能も準備されています。適当な数値を5つほど生成して、そのテストを行うコードは以下のようになります。

（あまり例がよくないですが・・・）

``` java
it.requires(5)
  .example(Generator.integersUpTo(1000))
  .toShow("税込み価格が取得できること", (expect, price) -> {
    expect.that(Sample.includeTax(price)).is((int)Math.floor(price * 1.08));
  });
```
requiresでランダムに準備する値の数を指定します。

exampleではどのようなテストデータを生成するかを指定します。ここではlambda-behaveで準備されたGenerator.integersUpToを使用しています。この他にも適当な文字列を生成するasciiStringsメソッドなどもあります

## 例外が発生することをチェック

引数にnullが渡された場合にNullPointerExceptionの発生をチェックすることも、JUnitと同様に可能です。

まず、includeTaxにnullチェックを記述します。

``` java
public class Sample {
    public static int includeTax(Integer price) {
        if(price == null) {
            throw new NullPointerException();
        }
        final float TAX_RATE = 0.08f;
        return (int)Math.floor(price * (1 + TAX_RATE));
    }
}
```

これをテストするlambda-behaveのコードは以下のようになります。

``` java
it.should("nullの場合ぬるぽ", expect -> {
    expect.exception(NullPointerException.class, () -> {
        Sample.includeTax(null);
    });
});
```

exceptionメソッドの第1引数に、発生する予定の例外を指定し、第2引数には例外が発生する処理をlambdaで記述します。

lambdaに慣れていないと少々書きづらいかもしれないですが、DSLちっくに書けるのはJavaっぽくなくてステキですよね！

こちらからは以上です。