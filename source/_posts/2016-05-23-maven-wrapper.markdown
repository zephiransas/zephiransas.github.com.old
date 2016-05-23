---
layout: post
title: "Maven Wrapperを使ってプロジェクトで使うMavenのバージョンを指定する"
date: 2016-05-23 16:34:12 +0900
comments: true
categories: Java
---

Javaでの開発において、ライブラリのバージョン管理にMavenを用いているところはたくさんあると思います。

しかし、pom.xmlを使って各ライブラリのバージョンを管理していても、各開発者が使うMavenのバージョンを固定することはできません。

プロジェクトで使うMavenのバージョンを固定したい！そんな場合に使えるのがMaven Wrapperです。

- takari/maven-wrapper -　https://github.com/takari/maven-wrapper

## 導入方法

導入方法は至って簡単。

maven wrapperを適用したいプロジェクトに移動して、以下のコマンドを発行するだけ。

``` bash
mvn -N io.takari:maven:wrapper
```

これだけで、プロジェクトに以下のファイルが追加されます。

- mvnw - Maven Wrapper経由でmavenを実行するためのファイル
- mvnw.cmd - mvnwのWindows版。Windowsで使う場合はこっちを使いましょう。
- .mvnディレクトリ - maven wraperがダウンロードしてきたMavenのバイナリとかが入ってる

上記のコマンドだと実行時の最新のバージョンが使用されるので、バージョンを指定したい場合はオプションで

``` bash
mvn -N io.takari:maven:wrapper -Dmaven=3.3.1
```

としてやりましょう。以降は今まで

``` bash
mvn clean
mvn package
```

としていたのをmvnwコマンドに置き換えるだけで

``` bash
./mvnw clean
./mvnw package
```

固定されたバージョンをMavenを利用することができます。

# .gitignoreの設定

Gitなどのバージョン管理にはmvnwとmvnw.cmdのみコミット対象とし、**.mvnディレクトリはコミット対象外**にしましょう。
