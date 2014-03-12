---
layout: post
title: "Java8で始めるLambda（基礎編）"
date: 2014-03-12 16:33
comments: true
categories: [Java, Java8, Lambda]
---

まもなくリリース予定のJava8。その中でも最も大きなインパクトを持つというProject Lambdaについて、ここ数日調べてみました。
今からLambdaをはじめようとする人向けに、何回かに分けてまとめてみたいと思います。

## インターフェースの宣言

まずは手始めに、引数で指定された文字列の前後に"["と"]"をつける処理を考えてみましょう。

Lambdaを使用するには、まずインターフェースを宣言する必要があります。上記の仕様から考えると

 - 引数はString型の引数が1つ
 - 戻り値もString型

となるので、この場合は以下の様なインターフェースを宣言します。

``` java LambdaInterface.class
public interface LambdaInterface {
  String method(String value);
}
```

ここで注意するべきことが1つ。Java8のLambdaとして使えるインターフェースには決まりがあるのですが、もっとも重要なのが**インターフェースのメソッドが1つだけ**ということです。インターフェースのメソッドが2つ以上ある場合には、それをLambdaとして使用することはできません。

これはLambdaの実装部分を書く際に、どのメソッドの内容を実装しているのかを書かないため、Lambdaを書いた場合は**そのインターフェースがもつ唯一のメソッド**に対して実装をおこなったとみなすからです。

一見、これは不便なように思ってしまうかもしれないですが、普段使うパターンのインターフェースのほとんどがjava.util.functionパッケージ内で用意されているので、実際にはそれ程不便ではありません。むしろ自分でインターフェースを用意するほうが稀かもしれません。

## Lambdaを使った記述

早速、先に宣言したインターフェースを使ってLambdaを書いてみましょう。Lambdaを記述する際の基本となる文法は、以下のようになっています。

``` java
[インターフェース名] [lambda式の名前] = (引数の型 引数,...) -> {
  （実装）
};
```

よってLambdaInterfaceを使って書くと、以下のようになります。

``` java Sample.class
public class Sample {

  public static void main(String... args) {
    LambdaInterface lambda = (String value) -> {
      return "[" + value + "]";
    };
    System.out.println(lambda.method("HOGE"));
  }

}
```
実行すると"[HOGE]"と出力されていることがわかると思います。

このように、匿名クラスを使った場合などと比べて、少ない記述量で実装できると思います。

## Lambdaの省略記法

また、このLambdaの記述では、以下のルールで、省略した記述を使用することもできます。

 - 引数の型は（型推論できるので）省略できる
 - 引数が1つの場合は、引数の()を省略できる
 - 但し、引数なしの場合は省略できない
 - 実装部分が1行の場合は、{}を省略可能。さらにreturn文も不要

上記ルールに沿ったLambdaであれば省略可能です。ですので先ほどのコードも以下の様に省略できます。

``` java Sample.class
public class Sample {

  public static void main(String... args) {
    LambdaInterface lambda = value -> "[" + value + "]";
    System.out.println(lambda.method("HOGE"));
  }

}
```

かなりスッキリしましたね！

## java.util.functionで提供されるインターフェースを使う

先ほど少し触れましたが、上記のLambdaInterfaceのような普段Lambdaとして使うインターフェースはjava.util.function内にいろいろ用意されています。
例えば、LambdaInterfaceの様に「String型の引数を1つ取り、String型の戻り値を持つ」インターフェースはjava.utl.functionパッケージ内にあるFunctionインターフェースを使います。

```
public interface Function<T, R> {
  R apply(T t);
}
```

最初の型引数Tには1つ目の引数の型、2つ目の型引数Rには戻り値の型を指定します。またapplyメソッドが実装対象となるメソッドです。

先のコードをFunctionインターフェースを使って書き直すと。

``` java Sample.class
public class Sample {

  public static void main(String... args) {
    Function<String, String> lambda = value -> "[" + value + "]";
    System.out.println(lambda.apply("HOGE"));
  }

}
```

となります。Functionの他にも

 - 引数を1つ持ち、戻り値がないConsumer
 - 引数がなく、戻り値があるSupplier
 - 引数を1つ持ち、戻り値がboolean型のPredicate
 - 引数を1つ持ち、かつこれが戻り値と同じ場合のUnaryOperator

などのインターフェースが提供されています。

どういった場合に、どのインターフェースを使えばよいかについては、Qiitaにまとめておきましたので、参考にしてみてください。

 - [Java8関数型インターフェース チートシート](http://qiita.com/zephiransas/items/3b03af4f9044df3182d0)

通常は、ここで提供されているインターフェースを使い、それ以外のパターンが発生した場合のみ、自分でインターフェースを宣言するほうがよいでしょう。

## メソッド参照を使う
メソッド参照もJava8で新しく追加された機能です。

メソッド参照を使うと、他のクラスのクラスメソッドやインスタンスメソッドを、Lambdaの実装として利用することができるようになります。メソッド参照はドット(.)でメソッドを呼ぶ代わりに、コロン2つで呼び出します。

```
[クラス名]::[メソッド名]
```

例えば先の文字列の前後にカッコをつけるメソッドが以下のようにSampleクラスのstaticメソッドとして定義されていた場合

``` java Sample.class
public class Sample {

  public static void main(String... args) {
    ...
  }

  private static String add(String value) {
    return "[" + value + "]";
  }

}
```

Sample.addをメソッド参照するには

``` java Sample.class
public class Sample {

  public static void main(String... args) {
    Function<String, String> lambda = Sample::add;
    System.out.println(lambda.apply("HOGE"));
  }

  private static String add(String value) {
    return "[" + value + "]";
  }

}
```

ここではstaticなクラスメソッドを使いましたが、インスタンスメソッドの場合も同様に、インスタンスを生成し、そこからメソッド参照をすることができます。

``` java Sample.class
public class Sample {

  public static void main(String... args) {
    Sample sample = new Sample();
    Function<String, String> lambda = sample::add;
    System.out.println(lambda.apply("HOGE"));
  }

  public static String add(String value) {
    return "[" + value + "]";
  }

}
```
