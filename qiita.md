# GraphQL ってどうやって使えばいいんだってばよ

前回の記事で開発環境が構築できたので、やれ開発だーと意気込んでいたところ、DB 設計で出鼻をくじかれました。
通常の DB ならスキーマ分類、テーブル定義、カラムのデータ型選択というように勝手がわかるのですが、QraphQL が間に挟まるとどうなるか想像もつきません。

というわけで、まずは GraphQL についてまとめることにしました。
本記事の目標は、GraphQL を[完全に理解した](https://mohritaroh.hateblo.jp/entry/2019/08/18/164000)と言えるようになることです。

## そもそも GraphQL って？

GraphQL は、クエリ言語(QL)の一種です。で、クエリ言語って？

ちょっと私のグーグル力に問題があるようで、以下にはあやふやな情報が含まれます。マサカリは随時受付中です。
クエリ言語(Query Language)は、データの問い合わせを行うための言語です。現在、普遍的に使われている SQL もその一種ですね。
また、最近では[Amazon も PartiQL というクエリ言語を発表したそうです](https://qiita.com/Iwark/items/f9da3d1c08b7e2b6437a)。

すなわち、今回取り上げる GraphQL もデータの問い合わせを行うための言語です。
SQL は RDBMS を対象にとってデータの問い合わせを行うことがメインの利用方法でしたが、GraphQL はサーバー-クライアント間通信をより効率的にしてくれるそうです。

### つまり、何がどう便利になるの？

API というと、[`REST API`](https://qiita.com/masato44gm/items/dffb8281536ad321fb08)が真っ先に浮かぶ人は多いと思います。私もそうです。
REST API と GraphQL の比較は[こちら](https://www.utakata.work/entry/2019/12/02/000000)でも行われていますが、一応ここでも軽く触れておきましょう。

REST API では、アクセスするリソースを URI で、リソースに対する操作をリクエストメソッドで規定していました。

例えば、以下のようなデータ構造が存在するとします。

[ER 図]

このとき、REST API では以下のようなアクセス方法になるかと思います。

[REST API 図]

この通り、リソースごとにエンドポイントが分割されており、その操作はリクエストメソッドという形で表現されます。
これは以下のメリットがあります。

- アクセスする対象のリソースがわかりやすい
- リソースに対する操作もリクエストメソッドという形で表現されているため、

一方で、以下のようなデメリットもあります。

- 1 度のリクエストにつき 1 つ(1 種類)のリソースしか取得できない
  - 複数のリソースを取得したい場合は何度もリクエストを送信する必要がある
- リソースの種類が増えるのに伴い、エンドポイントも増えていく

プロジェクトが肥大化するほど、上記のデメリットは無視できない大きさになっていきます。

GraphQL は、この問題を解消することができるのです。
例えば、先ほどの REST の図を GraphQL で置き換えてみましょう。

[GraphQL の図]

この通り、エンドポイントもリクエストメソッドも一本化されます。同じエンドポイントで、複数のリソースにアクセスすることができるようになるわけですね。
ちなみに、上記図では全部 POST ですが、GET でも問題なく動作するようです。
REST と比較して、GraphQL を使うことによるメリットは以下の通りです。

- 1 度のリクエストで複数のリソースを取得できる
  - 複数のリソースにまたがるデータを取得する場合でも、リクエストは 1 度で済む
- リソースが増えてもエンドポイントは 1 つのまま

デメリットについてはまだよくわかっていないので、ここでは一旦横に置かせていただきます。

ここまで調べて、ようやく GraphQL を採用するモチベーションが(なんとなく)わかってきました。誰だってリクエストを何回も投げるなんてしたくないですもんね。

### GraphQL を用いたリクエスト

ここで、基本的な GraphQL のリクエスト・レスポンスの流れについて確認していきます。参考資料は[公式のこちら](https://graphql.org/learn/serving-over-http/)。

#### GET リクエストの場合

ユーザーの一覧を取得する場合について考えてみましょう。このとき、ユーザー一覧を取得する GraphQL クエリは以下の文で表せるとします。

```graphql
{
  users {
    id
    name
  }
}
```

GET リクエストの場合、上記を圧縮してクエリ文字列に突っ込むだけです。

```uri
http://myapi/graphql?query={users{id,name}}
```

GraphQL クエリをクエリ文字列の query パラメータに入れるわけですね。クエリとクエリと query でごっちゃになりそうです。いや意味するところは同じなんですが。

#### POST リクエストの場合

POST の場合、2 つの方法があります。
1 つ目は JSON 形式で送信する方法です。当然ですが、ヘッダの`Content-Type`として`application/json`を指定します。

```json
{
  "query": "...",
  "operationName": "...",
  "variables": { "myVariable": "someValue", ... }
}
```

それぞれのフィールドに関する説明は以下の通りです。

- query: GraphQL クエリ本文(GET リクエストの query と同じ)
- operationName: 実行するクエリの名前(クエリを複数含む場合は必須)
- variables: query 中で使われる変数

常に必須なのは query のみであり、operationName と variables はオプショナルです。

JSON 形式とは異なる方法として、GraphQL のクエリ形式で送信する方法もあります。
`Content-Type`に`application/graphql`を指定すればいいようです。

#### レスポンスについて

レスポンスは基本的に以下の JSON 形式文字列で返されます。

```json
{
  "data": {...},
  "errors": [...]
}
```

言うまでもなく data はクエリ結果、errors はエラーです。
[GraphQL の仕様書](https://graphql.github.io/graphql-spec/)によると、正常に処理が実行できた場合は errors を含まないべきであり、また処理実行前にエラーが発生した場合は data を含まないべきであるそうです。

## GraphQL 詳細

ここからは、実際に GraphQL の仕様について確認していきます。基本的に[公式ページ](https://graphql.org/)の焼きなましです。

### 型について

GraphQL には型があります。

## 参考資料

[GraphQL のクエリを基礎から整理してみた](https://qiita.com/shunp/items/d85fc47b33e1b3a88167)
[0 から REST API について調べてみた](https://qiita.com/masato44gm/items/dffb8281536ad321fb08)
[REST を採用して気づいた「GraphQL って結局何が良いの？」について](https://www.utakata.work/entry/2019/12/02/000000)
[in between days](https://mohritaroh.hateblo.jp/entry/2019/08/18/164000)
[GraphQL 入門 - 使いたくなる GraphQL](https://qiita.com/bananaumai/items/3eb77a67102f53e8a1ad)
[GraphQL を最速でマスターするための意識改革３ヶ条](https://qiita.com/jabba/items/8d77ab86641937847673)
[Amazon が SQL++となる新しいクエリ言語、PartiQL をオープンソースで発表](https://qiita.com/Iwark/items/f9da3d1c08b7e2b6437a)
[GraphQL](https://graphql.org/)
