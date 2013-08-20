---
layout: post
title: "weinreでiOSのブラウザのデバッグを行う手順"
date: 2013-08-20 13:27
comments: true
categories: iOS
---
モバイル対応のWeb開発をしている時、iPhoneやiPadのブラウザでのデバッグに苦労したことはないですか？

PCの場合であれば、Chormeのデベロッパーツールとかでかなり簡単にデバッグを行うことができますが、iOSとかのSafariではデバッグツールをつかうことができません。

そんな時に便利なのが[weinre](http://people.apache.org/~pmuellr/weinre/docs/latest/)です。weinreを使うことで、iOS側で表示されたWebページのデバッグツールの情報を、PCから見ることができる大変便利なツールです。

今回はweinreの使い方を紹介します。

## weinreのインストール

まずはweinreをインストールします。weinreはnpmで提供されているので、以下のコマンドでインストールします。
npmをインストールしてない場合は、先に[node.jsをインストール](http://nodejs.org/)してください。

``` bash
$ sudo npm -g install weinre
```

自分の環境ではsudoを要求されたのですが、sudoなしでインストールできれば、それでもいいかと思います。


## weinreサーバの起動
次にweinreサーバを、以下のコマンドで起動します。

``` bash
$ weinre --boundHost 192.168.0.x
```
上記のboundHostにはlocalhostのIPアドレスを設定します。
weinreサーバはデフォルトの状態だとlocalhostからしか接続できません。この後iPad側から接続する必要がありますので、上記のようにboundHostを設定して、他のデバイスからも接続可能にしておく必要があります。

無事、起動できれば

``` bash
weinre: starting server at http://192.168.0.x:8080
```
と表示され、8080ポートでサーバが起動します。早速これをブラウザから見てみましょう。

![weinre_server](/images/20130820/weinre_server.png)

上記の画面が表示されれば、サーバが起動しています。

## iOSデバイスでブックマークレットの登録

次に、iOSデバイスでweinreサーバへ接続するためのブックマークレットを登録します。直接ブックマークレットを作れればいいのですが、iOSではそれができないようなので、ここでは新規にブックマークを作成して、これを編集することでブックマークレットを作成しています。

まずiOSデバイスのブラウザでweinreサーバ(ex: http://192.168.0.3:8080)を開きます。
そして、以下のjavascriptをクリップボードにコピーしておきます。

![bookmarklet](/images/20130820/bookmarklet.png)

次にiOSデバイスのブラウザで適当なWebページを開き、それをブックマークしておきます。

そしてそのブックマークを編集して、先ほどコピーしたjavascriptをペーストします。

![ios_bookmarklet](/images/20130820/ios_bookmarklet.png)


## デバッグの手順

以上で全ての準備が整ったので、実際にデバッグを行ってみます。

まずはPC側のブラウザでweinreサーバを開き、接続中のクライアント一覧を表示します。

![client](/images/20130820/client.png)

上記のリンクをクリックすると

![client_list](/images/20130820/client_list.png)

このような画面が表示されます。iOSデバイスから接続されると、上記のTargets一覧に表示されます。

iOSデバイス側からは、まずデバッグしたいWebページを表示し、その後、先に登録したブックマークレットを実行します。するとTargetが以下のようになります。

![connect](/images/20130820/connect.png)

あとはこの画面から、普通にデバッグツールを使用することができます。

![debug](/images/20130820/debug.png)

当然、デバッグツール側でHTML要素を変更してやれば、それがそのままiOSデバイス側のブラウザに反映されますので

![debug1](/images/20130820/debug1.png)

と変更してやれば、iOS側も

![debug2](/images/20130820/debug2.png)

といった感じで、即反映されます。

## 補足

今回はiOSデバイスしか手元になかったため未検証ですが、当然Androidでも同様にデバッグすることができます。
