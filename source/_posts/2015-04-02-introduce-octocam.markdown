---
layout: post
title: "GitHubのPull requestから、CHANGELOGっぽいものを作成するgemを作った"
date: 2015-04-02 17:31:42 +0900
comments: true
categories: Ruby
---

gemを作りました。名前はoctocamです。

- octocam - [https://rubygems.org/gems/octocam](https://rubygems.org/gems/octocam)

主な機能としては「GitHubから指定された日付期間にマージされたPull requestを抽出し、CHANGELOGっぽいMarkdownを生成する」というgemです。

定期的にリリースを行っている場合に、以前リリースされたときからどのような機能が増えたかをCHANGELOGとかに書き出しますが、そういった時に便利に使えると思います。

似たような機能を持つものはgemやnpmを探すと、結構あります。[この辺り](https://github.com/skywinder/Github-Changelog-Generator/wiki/Alternatives)とか。ただ、いずれも

- 日付の指定ができない。できたとしてもPull requestの作成日とか。
- issueやcommitを含めてしまう。
- Markdownで出力できない。
- 認証に対応してない。

などなど、要求を満たすものではなかったので、gemの作り方を勉強がてら作ってみました。

ワークフローとして、 **必ずPull requestでレビューをしてから、マージをおこなうワークフロー** を採用しているところであれば、フィットするように思います。

## インストール

以下のようにしてインストールします。

```
gem install octocam
rbenv rehash  # rbenvを使ってる人はrehash
```

もしプライベートなリポジトリにアクセスしたい場合は、[こちら](https://github.com/settings/applications)からPersonal access tokensを生成します。
あとは、生成したトークンを.bash_profileあたりから、環境変数「OCTOCAM_GITHUB_TOKEN」に設定しておきます。

```
export OCTOCAM_GITHUB_TOKEN="your-40-digit-github-token"
```

## 使い方

インストールされるとoctocamコマンドが使えるようになるので、以下のようにして実行します。

```
octocam -o zephiransas -r octocam -f 2015-01-01 -t 2015-01-31
```

-f,-tオプションにPull requestがマージされた日付を指定でききます。

** カレントディレクトリがgitのローカルリポジトリで、かつ、originがGitHubに設定されている場合であれば-o,-rオプションは省略できます。 **

欲しい機能ありましたら、issueを立てて頂くか、Pull requestを投げてください。
