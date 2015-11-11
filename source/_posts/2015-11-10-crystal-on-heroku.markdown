---
layout: post
title: "CrystalをHerokuで動かしてみた"
date: 2015-11-10 12:28:48 +0900
comments: true
categories: crystal, heroku
---

最近ちょっと話題のcrystal。これをHerokuで動かしてみました。

## Herokuの準備

まずはHerokuにアプリケーションを作成します。

Herokuでは当然crystalをサポートしていませんので、crystalのコンパイラを自前でインストールする必要があります。
Herokuにはこういったことを実現するために、buildpackという仕組みが用意されています。

crystal用のbuildpackは既にあるので、今回はこれを利用します。

- zamith/heroku-buildpack-crystal - https://github.com/zamith/heroku-buildpack-crystal

heroku createする際に上記のbuildpackを指定しておきます。

``` bash
heroku create --buildpack https://github.com/zamith/heroku-buildpack-crystal
```

ちなみに、これはあとから指定することも可能です。

``` bash
heroku
```

## アプリケーションの準備

次にcrystalで簡単なWebサーバを実装します。といっても[crystalの公式ページ](http://crystal-lang.org/)にあるサンプルに手を加えた簡単なものです。

app.crというファイル名で以下のファイルを準備します。

``` ruby app.cr
require "http/server"
require "option_parser"

server_port = 8080
OptionParser.parse! do |opts|
  opts.on("-p PORT", "--port PORT", "define port to run server") do |port|
    server_port = port.to_i
  end
end

server = HTTP::Server.new("0.0.0.0", server_port) do |request|
  HTTP::Response.ok "text/plain", "Hello world, got #{request.path}!"
end

puts "Listening on http://0.0.0.0:#{server_port}"
server.listen
```

注意するところは2点。

1点目は、option_parserを使って起動時のオプションでWebサーバのポート番号を指定できるようにしています。デフォルトでは8080ポートで起動します。

Herokuの場合、サーバのポートは$PORTの値を使用しなければいけませんので、起動時にその値を渡せるようにするためです。

2点目は、以下のようにServerのインスタンス生成時に"0.0.0.0"を指定することです。

``` ruby
server = HTTP::Server.new("0.0.0.0", server_port) do |request|
```

こうすることで、localhost以外からでもアクセス可能にしてあります。これを指定していない場合は、Herokuでの起動時に

``` bash
Error R10 (Boot timeout) -> Web process failed to bind to $PORT within 60 seconds of launch
```

というエラーが発生します。

できたら早速、起動してみましょう。

``` bash
crystal run app.cr
```

ブラウザからlocalhost:8080にアクセスし、以下の用に表示されれば、Webサーバが正しく起動しています。

![screen](/images/20151110/screen.png)

## その他のファイルの準備

次にProcfileを以下のように準備します。

``` bash Procfile
web: ./app -p $PORT
```

起動時に$PORTをWebサーバのポートとして、指定しています。

次にProjectfileを準備します。中身は空でOKです。

``` bash
touch Projectfile
```

これは本来は不要なファイルなのですが、crystalのbuildpack内でProjectfileがない場合は、crystalのアプリケーションとして認識してくれないため、空のファイルを作成しています。

## Herokuへデプロイ

app.cr、Procfile、Projectfileの3つが準備できたら、Herokuにデプロイしてみましょう。

``` bash
git init
git add .
git commit -m 'First commit'
heroku git:remote --app [APPNAME]
git push heroku master
```

あとは、Herokuにアクセスして、正しく動作していればOKです。

## まとめ

今回作成したコードはこちらにおいてありますので、参考にしてください。

https://github.com/zephiransas/crystal-heroku
