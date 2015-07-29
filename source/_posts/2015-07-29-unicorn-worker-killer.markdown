---
layout: post
title: "unicorn-worker-killerが便利だった件"
date: 2015-07-29 15:57:26 +0900
comments: true
categories: Ruby, Rails
---

自分が現在関わっているプロジェクトでは、nginx + unicornの構成で運用しているのですが、この構成でサーバのメモリが足りなくなるという現象に悩まされていました。

unicornのワーカプロセスは、通常では起動したままユーザからのリクエストを処理し、再起動されることはありません。
その関係で、長時間運用していると、そのワーカプロセスがメモリをあるだけ食いつぶすような挙動になります。

こんな時に便利なのが「[unicorn-worker-killer](https://github.com/kzk/unicorn-worker-killer)」です。

unicorn-worker-killerを使うことで、ワーカプロセスが以下の条件の場合に、自動的に再起動してくれます。

- ワーカプロセスが指定回数のリクエストを処理した場合
- ワーカプロセスが指定量のメモリを使用している場合

いずれの場合でもワーカプロセスの再起動は、現在のリクエストを処理した後に再起動（いわゆるgraceful restart）されます。

## 設定のしかた

設定はconfig.ruにて行います。

### リクエストの回数基準で再起動する

``` ruby config.ru
use Unicorn::WorkerKiller::MaxRequests, 3072, 4096
```

これはワーカプロセスが、3072回~4096回のいずれかの回数リクエストを処理したら再起動する設定です。

``` ruby config.ru
use Unicorn::WorkerKiller::MaxRequests, 3072, 4096, true
```

とすることで、unicorn.rbのstderr_pathで指定されたパスに状況を出力することができます。

### メモリの使用量を基準に再起動する

``` ruby config.ru
use Unicorn::WorkerKiller::Oom, (192*(1024**2)), (256*(1024**2)), 16
```

これはワーカプロセスが16回リクエストを処理する度に、自身のメモリ使用量をチェックし、これが192M~256Mのいずれかの使用量をオーバーしていた場合に、再起動する設定です。

### 設定が2つある理由

リクエスト回数とメモリ使用量の設定両方とも、しきい値を範囲で指定するようになっていますが、これには理由があります。

1つのしきい値だと、各ワーカが再起動するタイミングが、ほぼ同じになるからです。同じタイミングで全てのワーカプロセスが再起動してしまうと、その間リクエストを処理することができなくなってしまうので、これは好ましくありません。

ですので、しきい値を範囲で指定し、その範囲内のいずれかの値を実際のしきい値として採用するという仕組みになっています。

なので、しきい値の範囲は狭いより、広いほうが、ベターです。

## unicorn-worker-killerを試してみる
では、unicorn-worker-killerがちゃんとワーカプロセスをKillできているかを確認してみます。

シナリオとしては

- config/unicorn.rbのworker_processesは1として、ワーカプロセスは1つだけにする
- unicorn-worker-killerの設定は100回〜120回のリクエストを受けたタイミングで、ワーカプロセスを再起動するようにする

まずはGemfileに

``` ruby Gemfile
gem 'unicorn-worker-killer'
```

と設定します。config.ruの設定は、以下のようになります。

``` ruby config.ru
use Unicorn::WorkerKiller::MaxRequests, 100, 120, true
```

unicorn-worker-killerの詳細なログを出力するように設定しておきます。

この設定でunicornを起動します。すると以下ような感じでログが出力されます。

```
I, [2015-07-29T16:38:31.589102 #29745]  INFO -- : worker=0 spawning...
I, [2015-07-29T16:38:31.591242 #29745]  INFO -- : master process ready
I, [2015-07-29T16:38:31.593001 #29752]  INFO -- : worker=0 spawned pid=29752
I, [2015-07-29T16:38:31.593581 #29752]  INFO -- : Refreshing Gem list
I, [2015-07-29T16:38:47.035570 #29752]  INFO -- : worker=0 ready
```

pid=29752でワーカプロセスが1つ立ち上がりました。psで確認すると

```
zephiransas 29752 30.0  2.8 544708 235984 ?       Sl   16:38   0:15 unicorn worker[0] -c config/unicorn.rb -E production -D  
```

のような感じです。ここでブラウザから何度かアクセスすると、unicornのログに以下のように出力されます。

```
I, [2015-07-29T16:40:37.156111 #29752]  INFO -- : #<Unicorn::HttpServer:0x0000000343fa00>: worker (pid: 29752) has 119 left before being killed
I, [2015-07-29T16:40:37.349161 #29752]  INFO -- : #<Unicorn::HttpServer:0x0000000343fa00>: worker (pid: 29752) has 118 left before being killed
I, [2015-07-29T16:40:37.559274 #29752]  INFO -- : #<Unicorn::HttpServer:0x0000000343fa00>: worker (pid: 29752) has 117 left before being killed
I, [2015-07-29T16:40:37.649334 #29752]  INFO -- : #<Unicorn::HttpServer:0x0000000343fa00>: worker (pid: 29752) has 116 left before being killed
I, [2015-07-29T16:40:47.690545 #29752]  INFO -- : #<Unicorn::HttpServer:0x0000000343fa00>: worker (pid: 29752) has 115 left before being killed
```

pid=29752のワーカプロセスがあと何回リクエストを処理できるかが分かります。またリクエストするたびに1つづつ減っています。

では、引き続きブラウザからアクセスし、ワーカプロセスのメモリ使用状況を確認してみましょう。

```
zephiransas 29752  7.1  3.2 785940 269420 ?       Sl   16:38   0:26 unicorn worker[0] -c config/unicorn.rb -E production -D
```

メモリ使用量が少し増えているのがわかります。次にリクエストの残り回数を使い切り、ワーカプロセスが正しく再起動されるか確認します。

ブラウザからリクエストを投げ続けると、unicornのログに以下のように出力されます。

```
W, [2015-07-29T16:47:00.539055 #29752]  WARN -- : #<Unicorn::HttpServer:0x0000000343fa00>: worker (pid: 29752) exceeds max number of requests (limit: 119)
W, [2015-07-29T16:47:00.539621 #29752]  WARN -- : Unicorn::WorkerKiller send SIGQUIT (pid: 29752) alive: 383 sec (trial 1)
I, [2015-07-29T16:47:03.467363 #29745]  INFO -- : reaped #<Process::Status: pid 29752 exit 0> worker=0
I, [2015-07-29T16:47:03.467928 #29745]  INFO -- : worker=0 spawning...
I, [2015-07-29T16:47:03.472507 #7549]  INFO -- : worker=0 spawned pid=7549
I, [2015-07-29T16:47:03.473377 #7549]  INFO -- : Refreshing Gem list
I, [2015-07-29T16:47:19.137831 #7549]  INFO -- : worker=0 ready
I, [2015-07-29T16:47:20.251309 #7549]  INFO -- : #<Unicorn::HttpServer:0x0000000343fa00>: worker (pid: 7549) has 101 left before being killed
```

1行目でpid=29752がリクエスト回数上限の119回に達したことがわかります。

2行目ワーカプロセスに対してQUITシグナルを送信しています。

その後、別のワーカプロセスがpid=7549で起動しています。試しにpid=7549のメモリ使用量を見ると

```
zephiransas  7549  5.7  3.1 705752 261532 ?       Sl   16:47   0:18 unicorn worker[0] -c config/unicorn.rb -E production -D
```

となり、以前より減っていることがわかります。

Railsアプリを実運用するときには必須のgemだと思います。
