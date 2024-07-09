---
language: ja
title: Go x GraphQL でプロジェクト作成する際に考えたこと
published: 202107/31
description: ""
slug: "go-x-graphql"
---

## アーキテクチャについて

- clean archtecture like な設計を採用しまいた。。以下レポジトリを参考。 [bxcodec/go-clean-arch](https://github.com/bxcodec/go-clean-arch)
- 採用理由: 新規のプロダクトだったため、プロダクトの仕様が大きく変更されることが予想されました。そのため、こだわったというよりは、各Layerが多すぎず少なすぎないアーキテクチャがよいかなと思い、この設計にしました。

全体的な依存関係は以下です。

    presentation (gqlgenで自動生成 )
    ↑
    resolver  (gqlgenで自動生成 )
    ↓
    usecase interface
    ↑
    usecase implement
    ↓
    repository interface
    ↑
    repository implement(ORMはsqlboiler で自動生成)

## 命名規則

- どのレイヤーでも以下のようなPrefixを推奨。作成意図としては、命名で迷う時間を減らすため。
- 配列を返すAPI
  - 一覧: List
  - 絞り込み ListByXXXX
- 単一レコード返すAPI
  - Get
  - GetByXXX
- レコードを作成する
  - 単一レコードの作成: Create
  - 他のレコードも同時に作成: CreateWithXXX
- レコードを更新する
  - 単一レコードの更新: Update
  - 他のレコードも同時に更新: UpdateWithXXX

### 関数に pointer を渡すかどうかについて

以下を参照。

[公式](https://golang.org/doc/faq#methods_on_values_or_pointers)[公式 日本語訳](http://golang.jp/go_faq#methods_on_values_or_pointers)

[Go言語（golang）における値渡しとポインタ渡しのパフォーマンス影響について](https://medium.com/finatext/go%E8%A8%80%E8%AA%9E-golang-%E3%81%AB%E3%81%8A%E3%81%91%E3%82%8B%E5%80%A4%E6%B8%A1%E3%81%97%E3%81%A8%E3%83%9D%E3%82%A4%E3%83%B3%E3%82%BF%E6%B8%A1%E3%81%97%E3%81%AE%E3%83%91%E3%83%95%E3%82%A9%E3%83%BC%E3%83%9E%E3%83%B3%E3%82%B9%E5%BD%B1%E9%9F%BF%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6-70aa3605adc5)

    - 呼び出し元の値を変える必要があるなら、ポインタ
    - 引数（レシーバ）が大きければ、ポインタ
    - 引数（レシーバ）が小さければ、値

大きさについては、ぱっと検証することは不可能だと考え、渡した値に変更を加える場合はポインターを使い、変更を加えない場合は値渡しを使う方針

### DB 制約のチェックリスト

- 中間テーブルの uniq 制約
- 外部キー制約 FOREIGN KEY, REFERENCES

### データベースのモデリングについて

immutable data modeling を基本思想にする。 データの updateや deleteをすることはあまり推奨しない。

### テストについて

テストは mockはなるべく使用せずに行う。

GraphqQL の リクエストレベルのテストと、repository層のテストを中心におこなう。

[go のテストには cache がある](https://yyh-gl.github.io/tech-blog/blog/go-test-cache-clear/)

### リレーションは pointer で定義する

つかっている gqlgen の性質上、pointer で定義しておいた方が使いやすいため

[gqlgen で GraphQL サーバーを運用した感想](https://blog.ebiken.dev/blog/operating-graphql-server-with-gqlgen/)

### ログ設定について

- uber-go/zapを使用している
- 以下を参考に実施
  - [https://future-architect.github.io/articles/20200205/](https://future-architect.github.io/articles/20200205/ "https://future-architect.github.io/articles/20200205/")

### graphql の schema について

- github の graphql を参考にする
- 参考書籍 [https://github.com/vvakame/graphql-schema-guide](https://github.com/vvakame/graphql-schema-guide "https://github.com/vvakame/graphql-schema-guide")

## 参考資料 / その他メモ

### SQL Boiler 関連リンク

[SQL boilersample](https://github.com/ken-aio/go-echo-sqlboiler/)[gqlgen-sqlboiler-examples](https://github.com/web-ridge/gqlgen-sqlboiler-examples/tree/main/social-network)

### gqlgen関連 リンク

[sample code](https://github.com/oshalygin/gqlgen-pg-todo-example)[test sample](https://zenn.dev/konboi/articles/e59249d54651172e1caf)

[https://github.com/nutstick/gqlgen-clean-example](https://github.com/nutstick/gqlgen-clean-example "https://github.com/nutstick/gqlgen-clean-example")

[https://github.com/web-ridge/gqlgen-sqlboiler](https://github.com/web-ridge/gqlgen-sqlboiler "https://github.com/web-ridge/gqlgen-sqlboiler")[１回の Google ログインで、モバイルアプリ・自社サーバー・Google サービス・Firebase を連携させるフロー](https://qiita.com/eaglesakura/items/88a487be4f3b5249b6e6#firebase-auth%E3%81%AE%E5%88%B6%E9%99%90)

[sample of dataloder](https://github.com/graphql-go/graphql-dataloader-example/blob/master/main.go)[https://user-first.ikyu.co.jp/entry/go-graphql-dataloader](https://user-first.ikyu.co.jp/entry/go-graphql-dataloader "https://user-first.ikyu.co.jp/entry/go-graphql-dataloader")[https://github.com/hatena/go-Intern-Bookmark](https://github.com/hatena/go-Intern-Bookmark "https://github.com/hatena/go-Intern-Bookmark")

[MF Kessai Go + gqlgen を使った GraphQL アプリケーションサーバーの実装](https://tech.mfkessai.co.jp/2018/08/go-gqlgen-graphql/)

[many to many dataloader](https://w11i.me/graphql-server-go-part2-dataloaders)

[https://github.com/MichaelMure/git-bug](https://github.com/MichaelMure/git-bug "https://github.com/MichaelMure/git-bug")

### その他

[https://github.com/rabee-inc](https://github.com/rabee-inc "https://github.com/rabee-inc")

[リーダブルアーキテクチャ - usecase における時間軸と抽象度の統一](https://qiita.com/sonatard/items/2243bd6dcefa1b85dbda)
