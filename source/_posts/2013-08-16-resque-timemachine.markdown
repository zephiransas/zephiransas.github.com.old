---
layout: post
title: "TimeMachineで検証エラーが発生した場合の復旧方法"
date: 2013-08-16 14:57
comments: true
categories: Mac
---

我が家の環境では、[BuffaloのLS-XLシリーズ](http://buffalo.jp/product/hdd/network/ls-xl/)というNASを使って、MBAのTime Machine環境を作っているのですが、最近よくこんなメッセージが出るようになりました。

![error](/images/20130816/tm_error.png)

これはApple純正のTime Capsule以外をTime Machine環境に使用すると出てくるらしいです。
しかもこうなってしまうと、「後でバックアップを作成」にしても再度同じメッセージが表示されるし、「新規バックアップを作成」を選ぶと、それまでのバックアップの履歴は全て消えてしまいます。

なので、この場合の復旧手順をメモっておきます。

まずは、Time Machineを一旦「切」にしておきます。

![off](/images/20130816/tm_off.png)

次にNASをマウントして、そのマウントポイントまでのパスを確認しておきます。自分の場合は「/Volumes/share/unicorn.sparsebundle」でしたので、以降はこれで説明します。

次にターミナルを立ちあげ

``` bash
$ sudo chflags -R nouchg /Volumes/share/unicorn.sparsebundle
```
して、ロックを解除しておきます。次に

``` bash
$ hdiutil attach -nomount -noverify -noautofsck /Volumes/share/unicorn.sparsebundle
```
と入力します。実行すると以下のようなメッセージが表示されます。

``` bash
/dev/disk2 Apple_partition_scheme
/dev/disk2s1 Apple_partition_map
/dev/disk2s2 Apple_HFSX
```
この時の/dev/disk? の?部分は実行時の環境によって異なる数値が入っていますので注意。また、この次点でTime Machineイメージに対して修復処理がバックグラウンドで実行されています。そのログが/var/log/fsck_hfs.logに出力されています。復旧が終了したかどうかはそのログで確認するのでtailで眺めてやります。

``` bash
$ tail -f /var/log/fsck_hfs.log
```

おそらくこんな感じ。

``` bash
** Checking Journaled HFS Plus volume.
** Detected a case-sensitive volume.
   The volume name is Time Machine バックアップ
** Checking extents overflow file.
** Checking catalog file.
** Checking multi-linked files.
** Checking catalog hierarchy.
** Checking extended attributes file.
** Checking multi-linked directories.
** Checking volume bitmap.
** Checking volume information.
** The volume Time Machine バックアップ appears to be OK.
```

最後の「The volume Time Machine バックアップ appears to be OK.」が表示されればイメージの復旧は完了です。

終了したら、デタッチしておきます。

``` bash
hdiutil detach /dev/disk2s2
```

ただこの状態だと、イメージ内部ではまだ壊れたままという情報が残っているため、これを修正します。

``` bash
$ cd /Volumes/share/unicorn.sparsebundle/
$ vi com.apple.TimeMachine.MachineID.plist
```

とし、その内部の

``` xml
<integer>0</integer>
```

を

``` xml
<integer>2</integer>
```

に修正します。

これで全て完了です。Time Machineを「入」にして、バックアップを実行して、エラーが表示されなければ完了です。

え？根本的な解決になってない！ですって？　そりゃTime Machineを買うしかありませんがな！！ww

<iframe src="http://rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=zephiransas-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=ss_til&asins=B00DCM3W26" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>
