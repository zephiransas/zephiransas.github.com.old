---
layout: post
title: "Pusherを使ったChrome拡張を作る"
date: 2014-03-20 17:24
comments: true
categories: [Chrome,javascript,Pusher,WebScoket]
---

メッセージ等の新着通知やアップデート情報の配信など、アプリケーションへの通知の方法として、スマートフォンなどで使われるpush通知など、最近では様々なものがあります。

しかし自前で通知用のサーバを運用するのは手間がかかるので、これを簡単に使えるようにするサービスも増えてきました。例えば[Pusher.com](http://pusher.com)などがあります。これを使うことで、ブラウザへのリアルタイムな通知機能を、WebSocketを使って簡単に作成することができます。

![pusher-ss](/images/20140321/pusher-ss.png)

今回はPusherからの通知をChrome拡張で受信し、これをデスクトップ通知するサンプルを作成しましたので、解説してみます。

## Pusher側の設定

まずはPusher側に設定を行います。Pusherでアカウントを作成後、以下のようにアプリケーションを登録します。

![pusher1](/images/20140321/pusher1.png)

ここでEncryptionにチェックを入れておきましょう。チェックしなくても特に問題はないのですが、Chrome拡張の場合セキュリティの問題からSSLを使用したほうが、いろいろ都合がいいので、チェックするほうが無難です。

## Chrome拡張の作成

### manifest.jsonの設定

``` json manifest.json
{
    "manifest_version": 2,
    "name": "Pusher test extension",
    "version": "0.0.1",
    "description": "Pusher用 Chrome extension",
    "permissions" : [
        "notifications"
    ],
    "content_security_policy": "script-src 'self' https://stats.pusher.com; object-src 'self'",
    "background" : {
        "scripts": [
            "src/javascript/pusher.min.js",
            "src/javascript/background.js"
        ]
    }
}
```

今回はPusherからのメッセージをデスクトップ通知するようにしたいのでpermissionに
``` json
"permissions" : [
    "notifications"
],
```
を指定しています。

また、content_security_policyのscript-srcに **https://stats.pusher.com** を追加しています。httpではなく**https**を指定していることに注意してください。

``` json
"content_security_policy": "script-src 'self' https://stats.pusher.com; object-src 'self'",
```

このサイトは、Pusherのクライアントライブラリであるpusher.min.jsからアクセスしているのですが、これを許可していないとChrome拡張からPusherのサーバへ、正しく接続をすることができません。

次にpusher.min.jsですが、これは本来であればPusherのサイトに公開されているものを読み込んで使いたいところなのですが、Chrome拡張では外部のjavascriptを読み込むことができないようです。なので、ダウンロードしてソースに加えています。
ちなみにpusher.min.jsのホスト先は、[こちらで](http://pusher.com/docs/client_libraries)公開されています。


### background.js

あとはChrome拡張のバックグラウンドで実行されるbackground.jsで、Pusherとの接続を行います。

``` javascript background.js
var pusher = new Pusher("======== KEY ========", { encrypted: true });
var channel = pusher.subscribe('test_channel');
channel.bind('my_event', function(data) {
  var opt = {
    type: 'basic',
    title: data.title,
    message: data.message,
    iconUrl: ""
  }
  chrome.notifications.create("", opt, function(id){ /** Do Nothing */ });  
});
```

Pusherのコンストラクタに渡すkeyには、Pusherで作成したアプリケーションのKeyを指定します。

![pusher2](/images/20140321/pusher2.png)

また、大事なポイントとしてPusherのオプションに

```
{ encrypted: true }
```

を指定する必要があります。これは先のmanifest.jsonの設定でも触れましたが、この指定がない場合Pusherの接続先が **http://stats.pusher.com** となってしまいます。Chrome拡張ではhttpsでない外部サーバに接続することはセキュリティ上できませんので、上記のオプションを指定しています。

## Chrome拡張の読み込み

では、このようにして作ったChrome拡張を、Chromeに読み込ませて動作確認してみましょう。

ChromeのURL欄に chrome://extensions/ と入力します。まだデベロッパーモードを有効にしていない場合は「デベロッパーモード」をチェックします。次に「パッケージ化されていない拡張機能を読み込む」をクリックします。

![pusher3](/images/20140321/pusher3.png)

そして先ほど作ったChrome拡張を含むディレクトリを指定します。すると、拡張機能の一覧に表示されます。

![pusher4](/images/20140321/pusher4.png)

これでChrome拡張を導入することができました。

## Pusherからテストを行う

それでは実際にPusherから通知を行い、Chrome拡張に通知が表示されるかどうか確認してみましょう。

Railsなどを使って自分でサーバアプリを作ってもいいのですが、今回はPusherのEvent Creatorの機能を使ってみましょう。

Pusherの管理画面から「Event creator」を選択し、以下のように入力します。

![pusher5](/images/20140321/pusher5.png)

これで、Send eventボタンをおして、うまくいけば画面に以下の様な通知が表示されます。

![pusher6](/images/20140321/pusher6.png)

今回はアイコン画像のURLを指定しなかったので、なにも表示されていませんが、以下の用に画像へのURLを指定することでアイコンを表示することもできます。

``` javascript background.js
var opt = {
  type: 'basic',
  title: data.title,
  message: data.message,
  iconUrl: "http://example.com/path/to/icon"
}
```

## ソースなど

今回作成したChrome拡張は、こちらのGithubにまとめてあります。参考にしてみてください。

 - [pusher-extension](https://github.com/zephiransas/pusher-extension)

