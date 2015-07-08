---
layout: post
title: "論理削除とeager_loadでN+1問題が発生する件"
date: 2015-07-08 09:53:55 +0900
comments: true
categories: Ruby, Rails
---

Railsアプリにて論理削除とeager_loadを合わせて使うとN+1問題が発生することに気づいたのでメモ。

## N+1問題を確認する

まずはN+1問題が起きるようなモデルを作成します。よくあるブログアプリのような、ブログのエントリがあり、それにコメントが複数あるパターンです。

``` ruby
class Post < ActiveRecord::Base
  attr_accessible :title,
                  :content
  has_many :comments
end

class Comment < ActiveRecord::Base
  attr_accessible :post_id,
                  :name,
                  :content
  belongs_to :post
end
```

適当なデータを入れた後、これに対してrails cで以下のようにレコードを取得します。

``` ruby
Post.all.each do |post|
  puts post.comments.first.name
end
```

すると、以下のようなSQLが発行されます。

```
Post Load (0.1ms)  SELECT "posts".* FROM "posts"
Comment Load (0.1ms)  SELECT "comments".* FROM "comments" WHERE "comments"."post_id" = 12 LIMIT 1
ユーザ1
Comment Load (0.1ms)  SELECT "comments".* FROM "comments" WHERE "comments"."post_id" = 13 LIMIT 1
ユーザ1
Comment Load (0.1ms)  SELECT "comments".* FROM "comments" WHERE "comments"."post_id" = 14 LIMIT 1
ユーザ1
...
（以下続く
```

この場合は、**対象となったPostの件数分、CommentsテーブルへのSQLが発行されることになります。** これがN+1問題です。

## N+1問題に対処する

これを解決するには、eager_loadを使うことが一般的です。つまり

``` ruby
Post.eager_load(:comments).each do |post|
  puts post.comments.first.name
end
```

この場合のSQLは（一部簡略化しています）

```
SQL (0.2ms)  SELECT "posts".*, "comments".*
FROM "posts" LEFT OUTER JOIN "comments" ON "comments"."post_id" = "posts"."id"
```

となります。Postsテーブルと一緒にCommentsテーブルを取得しているので、SQLが1回だけ発行されていることがわかります。

美しい理想の世界です。ﾊﾗｼｮｰ

## paranoiaを導入する

さて本題。ここで**うっかり論理削除を導入**してみましょう。

Railsには論理削除に関するgemは多数ありますが、現在のデファクトスタンダードは[paranoia](https://github.com/radar/paranoia)だと思います。まずはparanoiaをGemfileに記述します。

``` ruby Gemfile
gem 'paranoia', '~> 1.0'  # Rails3系には1.0系を使用
```

その後、モデルを以下のように変更します。

``` ruby
class Post < ActiveRecord::Base
  acts_as_paranoid
  attr_accessible :title,
                  :content
  has_many :comments
end

class Comment < ActiveRecord::Base
  acts_as_paranoid
  attr_accessible :post_id,
                  :name,
                  :content
  belongs_to :post
end
```

その後、eager_loadしてみます。

``` ruby
Post.eager_load(:comments).each do |post|
  puts post.comments.first.name
end
```

するとSQLは以下の様に発行されます。（一部簡略化しています）

```
SQL (0.2ms)  SELECT "posts".*, "comments".*
FROM "posts" LEFT OUTER JOIN "comments" ON "comments"."post_id" = "posts"."id"
WHERE ("posts".deleted_at IS NULL)
```

PostsテーブルのWHERE条件にdeleted_at is nullが付与されているのは期待通りですが、Commentsテーブルには付与されていないので、これでは**論理削除されたCommentsテーブルの内容**も取得してしまいます・・・

では、以下のようにPostのcommentsにconditionsを付与するのはどうでしょう？

``` ruby
class Post < ActiveRecord::Base
  acts_as_paranoid
  attr_accessible :title,
                  :content
  has_many :comments, conditions: 'comments.deleted_at is null'
end
```

ここで同様にeager_loadするとSQLは以下の様に発行されます。

```
SQL (0.2ms)  SELECT "posts".*, "comments".*
FROM "posts" LEFT OUTER JOIN "comments" ON "comments"."post_id" = "posts"."id" AND comments.deleted_at is null
WHERE ("posts".deleted_at IS NULL)
```

WHERE条件が追加されて、なんだか、いい感じにeager_loadできました。

## 論理削除したデータも取得したい場合

さて、ここで少々頭がおかしくなって「削除したCommentも取りたい(^q^)」という気分になったとしましょう。

そこでPostクラスにcomments_with_deletedなるアソシエーションを追加します。

``` ruby
class Post < ActiveRecord::Base
  acts_as_paranoid
  attr_accessible :title,
                  :content
  has_many :comments, conditions: 'comments.deleted_at is null'
  has_many :comments_with_deleted,
           class_name: 'Comment',
           foreign_key: :id
end
```

さて、これを使ってeager_loadしてみましょう。

```
Post.eager_load(:comments_with_deleted).each do |post|
  puts post.comments.first.name
end
```

すると、以下のようなSQLが発行されます。

```
SQL (0.2ms)  SELECT "posts".*, "comments".* FROM "posts" LEFT OUTER JOIN "comments" ON "comments"."id" = "posts"."id" WHERE ("posts".deleted_at IS NULL)
Comment Load (0.1ms)  SELECT "comments".* FROM "comments" WHERE "comments"."post_id" = 12 AND ("comments".deleted_at IS NULL) AND (comments.deleted_at is null) LIMIT 1
ユーザ2
Comment Load (0.1ms)  SELECT "comments".* FROM "comments" WHERE "comments"."post_id" = 13 AND ("comments".deleted_at IS NULL) AND (comments.deleted_at is null) LIMIT 1
ユーザ1
Comment Load (0.1ms)  SELECT "comments".* FROM "comments" WHERE "comments"."post_id" = 14 AND ("comments".deleted_at IS NULL) AND (comments.deleted_at is null) LIMIT 1
ユーザ1
Comment Load (0.1ms)  SELECT "comments".* FROM "comments" WHERE "comments"."post_id" = 15 AND ("comments".deleted_at IS NULL) AND (comments.deleted_at is null) LIMIT 1
ユーザ1
Comment Load (0.1ms)  SELECT "comments".* FROM "comments" WHERE "comments"."post_id" = 16 AND ("comments".deleted_at IS NULL) AND (comments.deleted_at is null) LIMIT 1
```

最初のSQLでは、条件にcomments.deleted_at is nullが付与されていないので、これは期待通りなのですが、**その後、なぜかN+1問題が再発**しています。

現在のところ、これを回避できる方法は見つけられていません。

## 結論

結論を[社畜ちゃん](http://blog.oukasoft.com/OS/)にまとめていただきます。

![summary](/images/20150708/summary.png)

とは言っても論理武装が必要でしょうから、こちらも合わせてどうぞ。

- DELETE_FLAG を付ける前に確認したいこと。 - http://qiita.com/Jxck_/items/156d0a231c6968f2a474
- 論理削除が云々について - http://mike-neck.hatenadiary.com/entry/2015/03/24/231422
