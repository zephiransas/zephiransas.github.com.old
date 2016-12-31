---
layout: post
title: "スクフェス・ログサーバをつくった"
date: 2016-12-31 12:47:59 +0900
comments: true
categories: Java
---

今日は大晦日ですね。年末ですが今年も例によって、コード書いたりプラモ作ったり、普段の連休と同じくダラダラ過ごしております。

ところで今年の[ラブライブ！アドベントカレンダー](http://www.adventar.org/calendars/1360)はチェックしましたか？
自分も20日目に[劇場版ラブライブとμ’ｓメンバーのその後](https://zephiransas.goat.me/3OsFK6X7)というタイトルでエントリしてます。
今年はその他にも、さまざまな視点から見た素晴らしいエントリがたくさん集まってますので、ラブライバーならぜひチェックしてみてください。

で、22日目のエントリには[@hideo54](https://twitter.com/hideo54)さんの[スクフェスのライブスコアを取得する”schfeslog”を作った話](https://blog.hideo54.com/archives/591)というのがあります。
これはnodeで建てたプロキシを使って、スクフェスがサーバに送信してる通信内容をみて、ライブのプレイ結果をツイートすることができるツールです。

- hideo54/schfeslog - https://github.com/hideo54/schfeslog

これをみて「お、ツイートできるんなら、外部サーバにも送信できるんじゃね？」ってことで、早速コードを書いてみました。

まずはschfeslog側に外部サーバへの通信機能を実装しています。該当するPull Requestは[こちら](https://github.com/hideo54/schfeslog/pull/4)。単純にプレイデータをJSON形式にして、設定でされたサーバにPOSTするだけです。

これを受信するサーバはこちら。

- zephiransas/schfeslogsvr - https://github.com/zephiransas/schfeslogsvr

送信されたプレイデータを一覧で見ることもできます。ちなみに私のプレイデータがこちら

- schfeslog - http://schfeslog.herokuapp.com/

見た目とかは、もうちょっと改善したいところです・・・

最近、ちょっとJavaの案件をやってるせいもあって、真面目にSpring Bootで書いています。こういったRESTなアプリケーションを作るにはSpring Bootはとても簡単でいいですね。

サーバ側は簡単に自分用に環境を作れるよう、Deploy to Herokuボタンも準備してますので、興味のあるかたはschfeslogと一緒に、ぜひ試してみてください。
