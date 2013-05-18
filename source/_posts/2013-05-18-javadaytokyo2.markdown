---
layout: post
title: "Java Day Tokyo 2013に参加してきた（セッション）"
date: 2013-05-19 00:17
comments: true
categories: [Java,勉強会]
---

前回の基調講演につづいて、今回は自分が参加したセッションのレポートです。

## Ask the Experts

基調講演にも登壇したJavaSE,JavaFX,JavaEEの各キーパーソンに、直接質問できるセッションです。ちょうど昼の時間だったので、本来は昼食をとりつつ受けるセッションなんですが、うっかり自分は昼食を仕入れるのを忘れてました・・・

JavaSEの担当は、Simon Ritter氏

<img src="http://farm8.staticflickr.com/7288/8740173841_d492ab6090_n.jpg"/>

JavaFXの担当は、Jim Weaver氏

<img src="http://farm8.staticflickr.com/7285/8741292898_42456c92d5_n.jpg"/>

JavaFXの担当は、Arun Gupta氏

<img src="http://farm8.staticflickr.com/7281/8741293406_f87d358552_n.jpg"/>

通訳の方が居たので、もちろん日本語で質問できたのですが、意外と質問が少ないような感じでした。


印象に残ったのは、最後のJavaEEの質問で

- Q「JavaEEはいろんな仕様で構成されているが、最低限どれを使ってればJavaEEを使ってることになるのか？」

という質問がありました。（ちょっと意地悪な質問な気もしますがw）それに対してArunは

- A「難しい質問だけど、強いて言えばServletかな？あ、でも、Strutsはダメだね！」

と返して、会場は大爆笑でしたw

## Javaプラットフォームにおける Batch アプリケーション (JSR 352)

JavaEE7で導入されるJBatchのセッション。担当はArun氏です。

<img src="http://farm8.staticflickr.com/7286/8741295358_cce2037082_n.jpg"/>

このJBatchは、エンタープライズアプリケーションでよくある、バッチ処理を効率よく書くためのフレームワークです。セッションを受けつつTLを眺めてて知ったのですが、どうもSpring Batchと非常によく似た仕組みのようです。設定をXMLで記述していくのが少々面倒な気もしますが、この辺りが改善されていけば、バッチ処理のスタンダードになるかなーという感じ。JavaEE準拠のサーバはもちろん、スタンドアロンの環境でも実行可能との事。

## エンタープライズ環境における並列処理の実装方法について

我らの王子こと、寺田さんによるConcurrency Utilityのセッション。

以前からJavaではThreadを使って、並列処理を書くことが可能でしたが、実際にはなかなか難しいものでした。これを簡単に行えるのがConcurrency Utilityです。実際に以下のデモを会場で見せていました。

<iframe width="560" height="315" src="http://www.youtube.com/embed/s9OB3lDPwtg?rel=0" frameborder="0" allowfullscreen></iframe>

詳細は[寺田氏のブログ](http://yoshio3.com/2013/05/15/concurrency-utilities-for-ee-7/)に詳しいですが、Concurrncy Utilityを使うことで、CPUリソースを無駄なく使うコードを、簡単に実装することができるようになります。

## Java the Night

<img src="http://farm8.staticflickr.com/7285/8740181743_e4b88c0b29_n.jpg"/>

最後はお楽しみ（？）のJava the Night。日本のJava界を代表するエンジニアがLT&デモを行うという趣向です。
一人8分の持ち時間でした。どのLTもさすがはJava界のスーパーエンジニア！と唸らせる、最高に面白い内容でした。
前回のJavaOne Tokyoの時もそうでしたが、この最後のLT枠に参加せずして、Javaのイベントに参加したとは言えないくらい、充実した内容です。
なかでも印象に残ったのは、北海道の大学生2人。

<img src="http://farm8.staticflickr.com/7283/8741304934_655e6abe63_n.jpg"/>

プレゼンソフトなんですが、JavaFXで様々なエフェクトをつけることができるというものでした。彼らの初々しい（！）発表を聞きながら、その将来に期待をするとともに、まだまだ自分も頑張らなければ、と想いを新たにしました。

## JavaSE7 日本語ドキュメント提供開始

そしてJava the Night終了後、寺田さんから重大な発表がありました。それはJavaSE7の日本度ドキュメントの提供を開始した、とのアナウンスでした。JavaSE7リリース後、しばらくしても日本語ドキュメントが提供されていませんでした。様々な方からも要望は上がっていましたが、残念ながら日本語ドキュメントは提供されないという決定がなされました。

<blockquote class="twitter-tweet" lang="ja"><p>影響度が大きい事を十分承知で申し上げます。誠に残念ながら直近で、提供の予定はございません。“@<a href="https://twitter.com/skrb">skrb</a>: @<a href="https://twitter.com/yoshioterada">yoshioterada</a> それよりもJava SE 7の日本語のJavadocはリリースされないのでしょうか？”</p>&mdash; Terada Yoshioさん (@yoshioterada) <a href="https://twitter.com/yoshioterada/status/183876594010558464">2012年3月25日</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

その後、有志を中心にボランティアで日本語化しようかといった動きもありましたが、本格化するには至りませんでした。

そして、Java Day Tokyoにてサプライズとでもいいましょうか、[JavaSE7の日本語ドキュメント提供開始](http://yoshio3.com/2013/05/14/%EF%BD%8A%EF%BD%81%EF%BD%96%EF%BD%81-%EF%BD%93%EF%BD%85%EF%BC%97%EF%BD%81%EF%BD%90%EF%BD%89%E6%97%A5%E6%9C%AC%E8%AA%9E%E7%89%88%E6%8F%90%E4%BE%9B%E9%96%8B%E5%A7%8B/)のアナウンスがされました。

JavaSE7がリリースされて随分経ちますが、これでようやくスタートライン。ようやくJava7を普通に使ってもらう環境が、日本でも整ったといった感じです。いろいろ想いはありますが、日本オラクル＆寺田さんの努力に敬意を表したいと思います。

## Conclusion

<img src="http://farm8.staticflickr.com/7286/8740206373_002b31196d_n.jpg"/>

以上、2回に分けてJava Day Tokyo 2013の模様をまとめました。

1年ぶりにJavaのカンファレンスに参加して「自分はやっぱり、このJavaのコミュニティが好きなんだなぁ」という思いを強くしました。もちろん他のコミュニティでも似たような経験はできるとは思いますが、古参から新参まで多くのカラーが集まり、多種多様な人がいるコミュニティも珍しいように思います。そのような環境でエンジニアをやれるのは、とても嬉しいことです。

さて、岡山Javaユーザ会でも、Java Day Tokyo 2013報告会@岡山と題して、報告会をやります。

お申込みは[こちら](http://local.aguuu.com/events/15432)からです。

お近くの方は、是非ご参加ください。

