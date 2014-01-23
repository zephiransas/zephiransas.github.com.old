---
layout: post
title: "werckerでrspecとcucumberのCI環境を作る"
date: 2014-01-23 14:30
comments: true
categories: Ruby
---

去年作った[Photo Leaf](http://www.photo-leaf.com/)というWebサービスがあるのですが、これのCI環境を作成したいなぁと思っていたところwerckerというCIサービスを使って構築できたので、そのまとめです。

## werckerとは？

[wercker](http://wercker.com/)は[TravisCI](https://travis-ci.org/)などに代表されるように、外部でビルド（というかテスト）やデプロイを行ってくれるCI（継続的インテグレーション）サービスです。

使い方としては、リモートリポジトリに変更がpushされた時点でこれをフックして、CIサービス側でテストしたり、場合によってはそのまま本番環境に自動デプロイとかもしてしまえば、変更内容を安全、かつ迅速にリリースできる仕組みが出来上がるわけです。イマドキっぽいですね！

werckerの特徴としては

#### GithubとBitbucketに対応

TravisCIはGithubにあるリポジトリしかビルド対象にできません。Githubのサービスなんだから、当たり前といえば当たり前ですが・・・

#### （今のところ）無料

2014/1/23現在はベータのようです。将来的にはどうなるのかわからないですが、今のところ無料で使えます。制限としては「**1つのビルドが25分以内に完了すること**」ぐらいです。エンタープライズなシステムだとキツイかもですが、そこそこの規模なら問題ないんじゃないでしょうか。
 
#### テストを実行するための仮想環境が豊富に用意されている

werckerでビルドを実行する際にはBoxという仮想環境内で実行されます。これが予め様々な種類が用意されています。通常のRuby(Rails)の環境とかだけではなく、JavaやAndroidといった環境も用意されています。また[Boxを自分で作る](http://devcenter.wercker.com/articles/boxes/)こともできるようです。

#### privateリポジトリもビルドできる

TracisCIは無課金だとprivateリポジトリはビルドできません。しかしwerckerはprivateリポジトリをビルドできます。
Photo LeafのソースはBitbucket上のprivateリポジトリで管理しているため、今までCIサービスを利用することができなかったのですが、werckerはprivateリポジトリでもビルドできるので便利です。

## wercker.ymlの設定

Photo Leafでは、テストをrspecとcucumberで書いています。cucumberではjavascriptのドライバとしてcapybara-webkitを使ってます。そのためwerckerで動かすには設定が若干面倒です。

werckerにログインして、とりあえず普通にビルドするまでの手順は、以下の記事に詳しいのでこちらを参照してください。

 * [Githubのプライベートリポジトリでも無料で使えるCI、Werckerを使ってrails newからHerokuのデプロイまでやってみる](http://blog.mah-lab.com/2014/01/08/rails-wercker-heroku-deploy/)

上記で設定したwercker.ymlに対して、rspecとcucumberを実行するように設定していきます。自分が設定したwercker.ymlは以下のような感じ

``` yml wercker.yml
box: wercker/rvm
# Build definition
# See the Rails section on the wercker devcenter:
# http://devcenter.wercker.com/articles/languages/ruby/settingup-rails4.html
# You will want to define your database as follows:
services:
    - wercker/postgresql
# See more about services on our devcenter:
# http://devcenter.wercker.com/articles/services/

build:
    steps:
        # Uncomment this to force RVM to use a specific Ruby version
        # - rvm-use:
        #       version: 2.1.0
        - script:
            name: Make tmp directory
            code: mkdir tmp

        - script:
            name: Enable virtual display
            code: |-
              # Start xvfb which gives the context an virtual display
              # which is required for tests that require an GUI
              export DISPLAY=:99.0
              start-stop-daemon --start --quiet --pidfile /tmp/xvfb_99.pid --make-pidfile --background --exec /usr/bin/Xvfb -- :99 -screen 0 1024x768x24 -ac +extension GLX +render -noreset

              # Give xvfb time to start. 3 seconds is the default for all xvfb-run commands.
              sleep 3
        # Install (apt-get) packages
        - install-packages:
            packages: libqtwebkit-dev
        # A step that executes `bundle install` command
        - bundle-install

        # A step that prepares the database.yml using the database in services
        - rails-database-yml:
            service: postgresql

        # A custom script step, name value is used in the UI
        # and the code value contains the command that get executed
        - script:
            name: echo ruby information
            code: |
                echo "ruby version $(ruby --version) running"
                echo "from location $(which ruby)"
                echo -p "gem list: $(gem list)"

        - script:
            name: Set up db
            code: RAILS_ENV=test bundle exec rake db:schema:load

        - script:
            name: Run RSpec
            code: bundle exec rspec spec

        - script:
            name: Run Cucumber
            code: bundle exec cucumber features

```

* 16~18行目 - テスト用のtmpディレクトリを作成（rspecのテストでtmpを使ってるので、必要なければ不要です）
* 6~7,37~38行目 - テスト用のPostgreSQLを実行する
* 53~55行目 - rspecを実行
* 57~59行目 - cucumberを実行

## capybara-webkitを動かすための注意点

capybara-webkitを動かすにはX11が使用できる必要があります。なのでXvfbを仮想環境で実行する必要があります。その設定が以下の部分。

``` yml wercker.yml
- script:
    name: Enable virtual display
    code: |-
      # Start xvfb which gives the context an virtual display
      # which is required for tests that require an GUI
      export DISPLAY=:99.0
      start-stop-daemon --start --quiet --pidfile /tmp/xvfb_99.pid --make-pidfile --background --exec /usr/bin/Xvfb -- :99 -screen 0 1024x768x24 -ac +extension GLX +render -noreset

      # Give xvfb time to start. 3 seconds is the default for all xvfb-run commands.
      sleep 3
# Install (apt-get) packages
- install-packages:
    packages: libqtwebkit-dev
```

上記の設定で仮想環境上でXvfbを実行できます。

## まとめ

- werckerはGithubとBitbucket両方に対応してるよ！
- werckerは無料でprivateリポジトリもビルドできるよ！
- capyabara-webkitを使うときにはXvfbを実行してよ！