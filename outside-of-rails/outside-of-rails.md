# OSS にみる レールの外側

author
:  竹内雄一

institution
:  Takeyu Web Inc.

allotted-time
:  20m

# @takeyuweb

- 2008年〜フリーランス
- 2016年 法人成り
- Rails おじさん（Rails 1.0系～）
- Saitama.rb発起人

# Takeyu Web Inc.

![](logo.png){:relative_width='100'}

# Takeyu Web Inc.

フルリモートとフルフレックスにこだわってるRails受託会社

# リモートワークとRails

リモートワークではコミュニケーションの敷居が高い

レールがあることは、リモートに向いている（と、思っている）

# レールに乗る

他人（過去の自分含む）にコードを読むヒントを与えることができる

# そんなレールから外れるということ

- 『常識』が通用しなくなる？
- 世界中の開発者が参加するOSSではどうやっているか？
- リモートワークのヒントもあるのでは

# 今回見たOSS

| Name | Stars | Contributors |
| ---- | ----- | ------------ |
| GitLab | 21,029 | 1,545 |
| Discourse | 25,573 | 649 |
| Mastodon | 14,031 | 476 |

# 見ること

- appの中身
    `rails new` したときデフォルトで作られるもの以外
- その他

# GitLab

https://github.com/gitlabhq/gitlabhq

- 21,029 Stars
- 1,545 Contributors

# GitLab のapp (1)

- finders
- graphql
- policies
- presenters
- serializers

# GitLab のapp (2)

- services
- uploaders
- validators
- workers

# GitLab のapp

- いっぱいある！
- 独自のディレクトリについて `README.md` がしっかり書いてある


# GitLab の `xxxx/README.md`

- 読めば使い方はわかる
- Wikiなど外部リソースへ誘導されない
- 必要なときにすぐ存在を見つけられて良い

# `xxxx/README.md` に書いてあること

- いつ使うか？
- なんで標準のじゃアカンのか
- 具体的に何が嬉しいか
- コードの書き方

# `app/presenters/README.md`

[app/presenters/README.md](https://github.com/gitlabhq/gitlabhq/tree/f3f1df1476ba7fe223e5d8d6707a7675dc9fa597/app/presenters)

# GitLab lib

gemで提供されるようなものなど、特定の目的のために使うクラス群。こちらは README.md はない

- api Grape::API
- backup バックアップ処理いろいろ database / file etc
- banzai Markdown => HTML renderer etc...

# GitLab その他

- Migration の spec
- 独自Cops
- 整理された `routes`

# Migration の spec 

`db/migrations/*.rb` ごとに実行前後どうなるべきか？の spec がある

[db/migrate/20170713104829_add_foreign_key_to_merge_requests.rb](https://github.com/gitlabhq/gitlabhq/blob/master/db/migrate/20170713104829_add_foreign_key_to_merge_requests.rb)
[spec/migrations/add_foreign_key_to_merge_requests_spec.rb](https://github.com/gitlabhq/gitlabhq/blob/master/spec/migrations/add_foreign_key_to_merge_requests_spec.rb)


# 独自Cops

プロジェクトでやってはいけないこと、やるべきことについてCopsで指摘してれる

# `RuboCop::Cop::GitLab::FinderWithFindBy`

`rubocop/cop/gitlab/finder_with_find_by.rb`

```ruby
# 通常こうやって #execute を使うんだけど
results = DummyFinder.new(some_args).execute

# find_by と組み合わせて使うときは
DummyFinder.new(some_args).execute.find_by!(1)
# こう書くようにしようね
DummyFinder.new(some_args).find_by!(1) 
```

# 独自Cops の spec

`spec/rubocop/**_spec.rb`

独自Copsが具体的になにを警備しているかわかりやすくてよい

# 整理された routes (1)

`config/routes.rb`

```ruby
Rails.application.routes.draw do
  # (snip)
  draw :api
  draw :sidekiq
  draw :help
```

# 整理された routes (2)

`config/routes/api.rb`

GraphQL と REST API

```ruby
constraints(::Constraints::FeatureConstrainer.new(:graphql)) do
  post '/api/graphql', to: 'graphql#execute'
  mount GraphiQL::Rails::Engine, at: '/-/graphql-explorer', graphql_path: '/api/graphql'
end

API::API.logger Rails.logger
mount API::API => '/'
```

# GitLab のまとめ

- 「うちではこう書いてください」みたいなの、多い
    - 理由と書き方について詳しく説明があるので辛くはない
- Rails標準におけるActiveRecordの書き方みたいに、GitLabのFinderの書き方のような決まり事がはっきりしている
- 独自Copで決まり事を守っているか自動チェックしててすごい

# GitLab のまとめ

意識高い（系ではない）印象

# Discourse 

https://github.com/discourse/discourse

- 25,573 Stars
- 649 Contributors

# Discourse の app (1)

- serializers
- services

# Discourse の app (2)

- `app` を見ても 「ウッ」ってならない
- 標準のようでそうでもないものもある
    - `app/jobs`

# Discourse の app/serializers

- active_model_serializers gem

# Discourse の app/jobs (1)

- ActiveJobではない
- `include Sidekiq::Worker` した共通の基底クラスを持つクラス群
    - それなりに厚い
    - 独自の仕組み
        - `Job::Base#execute`

# Discourse の app/jobs (2)

- readonly modeへの対応
- onceoff

# Discoures の app/services

自由に作られている印象。基底クラスや `.new.execute` のような決まりはない。カオス。

- UserMerger
- SearchIndexer
- HandleChunkUpload
- DestroyTask

# Discourse の lib

- scheduler 
- validators
- wizard
- auth/authenticator etc

# Discourse の その他

- Danger

# Danger

- Danger http://danger.systems/
- CI上で動作、PRに対して自動で指摘
- PR自体の問題「テスト書いてないけど？」「こんなのレビューできないよ」みたいなの
- ルールをRubyで書ける（Dangerfile。次ページ。）

# Discourse の Dangerfile （抜粋）

```
if git.lines_of_code > 500
  warn("This PR seems big, we prefer smaller PR. Please be sure this is needed and can't be split in smaller PRs.")
end
```

# Discourse のまとめ

- 「うちではこう書いてください」みたいなのはあまりない
- なるべくレールに載る意識を感じる
- `app/services` にモヤモヤ

# Mastodon

https://github.com/tootsuite/mastodon

- 14,031 Stars
- 476 Contributors

# Mastodon の app

- chewy
- lib
- policies
- presenters
- serializers
- services
- validators
- workers

# Mastodon の app/chewy

- chewy gem
    - Elasticsearch

# Mastodon の app/lib

- なぜ `Rails.root.join("lib"")` でないのか？よくわからない
    - `lib` はほとんど使っていない

# Mastodon の app/policies

- 権限管理
- pundit gem

# Mastodon の app/presenters

- `ActiveModelSerializers::Model` だったりそうでなかったり
- シリアライズに使ったり、ActiveRecordモデル間の関係（フォロワー、ブロック etc..）を表示用にまとめたり

# Mastodon の app/serializers

- active_model_serializers gem

# Mastodon の app/services

パブリックなインスタンスメソッドは`#call`のみ

- `class HogeService < BaseService`
- `HogeService.new.call(arg1, arg2, arg3)`

# Mastodon の app/workers

- `Sidekiq::Worker`

# Mastodon の まとめ

- `app/services` は読みやすいし、使い方がはっきりしていて心理的負担が少ない
- が、他はなんだかよくわからん
- `bundle install` がなかなか通らなくてつらい

# 余談：脱線事故（僕の失敗）

記事で読んだから試しに使ってみた

- あとから別にいらなかった
- そもそも正しく使えてない
- ルールが明確でなく各自自由に書いちゃう→カオス
- 依存gemが増えてアップデート辛い

# まとめ

- フレームワーク的に使ってほしいものは `app` 直下、そうでないものは `lib`
- 本当に必要か？
- 新しいレールを敷く
- 人は弱いので RuboCop Danger などツールの力を借りる

# 宣伝（1）

Saitama.rb やってます！

- 埼玉県在住のRubyist
- 毎月第3土曜日 13:00～
- 大宮駅東口徒歩1分
- 初心者から（ときどき）Railsコミッターまで

# 宣伝（2）

Rails受託案件募集してます！

- 11月～ぐらい
- 週3～
- リモート
- 準委任（条件はご相談ください）
