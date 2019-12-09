# GraphQL ってどうやって使えばいいんだってばよ

前回の記事で開発環境が構築できたので、やれ開発だーと意気込んでいたところ、DB 設計で出鼻をくじかれました。
通常の SQL と RDBMS を使った構築ならある程度勝手がわかるのですが、QraphQL が間に挟まるとどうなるか想像もつきません。

というわけで、まずは GraphQL についてまとめることにしました。
本記事の目標は、GraphQL を[完全に理解した](https://mohritaroh.hateblo.jp/entry/2019/08/18/164000)と言えるようになることです。

## そもそも GraphQL って？

GraphQL は、クエリ言語(QL)の一種です。で、クエリ言語って？

ちょっと私のグーグル力に問題があるようで、以下にはあやふやな情報が含まれます。マサカリは随時受付中です。
クエリ言語(Query Language)は、データの問い合わせを行うための言語です。現在、普遍的に使われている SQL もその一種ですね。最近では[Amazon も PartiQL というクエリ言語を発表したそうです](https://qiita.com/Iwark/items/f9da3d1c08b7e2b6437a)。
すなわち、今回取り上げる GraphQL もデータの問い合わせを行うための言語です。

GraphQL には、上記のデータ問い合わせを行うクエリ言語とは別に、データの構造を定義するスキーマ言語をもちます。
スキーマ言語によってデータ構造の型定義を行うことができます。この型定義は Web API の通信時に用いられ、型安全なクライアント-サーバー間通信が可能になるそうです。

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

GraphQL クエリをクエリ文字列の query パラメータに入れるわけですね。クエリとクエリと query でごっちゃになりそうです。

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

### スキーマ言語について

GraphQL には型があります。この型定義を行うことができるのがスキーマ言語です。
あらゆる GraphQL を介した通信は、このスキーマ言語による型定義を元にしています。

#### スカラー型

スカラー(scalar)とは、物理学において大きさのみを持つ量のことだそうです[(Wikipedia 調べ)](<https://ja.wikipedia.org/wiki/%E3%82%B9%E3%82%AB%E3%83%A9%E3%83%BC_(%E7%89%A9%E7%90%86%E5%AD%A6)>)。
QraphQL においては、数値や文字などの実データを表す型として用いられるようです。

まず、QraphQL においてデフォルトで定義されている型を紹介します。

- Int: 符号付きの 32 ビット整数
- Float: 符号付きの倍精度浮動小数点数
- String: UTF-8 の文字列
- Boolean: `true`もしくは`false`
- ID: オブジェクトの再取得やキャッシュのキーとして用いられる、一意な識別子
  - この型は同様の手段で文字列にシリアル化される
  - この型をフィールドに与えるということは、そのフィールドは人間が読むためのものではないことを意味する

ID だけちょっと特殊ですが、他の型はよく見る型ですね。
GraphQL は上記以外にも、ユーザー独自に型を定義することができます。`Date`という型を定義したい場合はこんな感じ。

```graphql
scalar Date
```

ただし、独自定義したスカラー型について、ユーザーは必要に応じてその変換処理を記述しなければなりません。上記`Date`型で言うのであれば、年月日までなのか、秒あるいはミリ秒までなのか、フォーマットはどうするかなどについて記述するべきでしょう。
ただし、このスカラー型の変換については GraphQL のスキーマ定義範囲外です。GraphQL ライブラリによって異なる記述になるので、利用するライブラリのドキュメントを確認してみましょう。たとえば`gqlgen`なら[このページ](https://gqlgen.com/reference/scalars/)です。

#### オブジェクトの型とフィールド

ここでは基本的な型の記述について確認していきましょう。
GraphQL の型定義は、型名とフィールドからなります。

```graphql
type Character {
  name: String!
  appearsIn: [Episode!]!
}
```

上記の例について確認してみましょう。

- `Character`というオブジェクトの型を定義している
- `name`と`appearsIn`の 2 つのフィールドを持つ
- `String`は文字列のスカラー型
  - `String!`と最後に`!`をつけることで、そのフィールドが null にならないことを示す
- `Episode`はユーザー定義の型
  - `[Episode]`と`[]`で囲むことにより、配列であることを示す
  - `!`がついているので、このフィールドは null にならないことを示している

直感的でわかりやすいかと思います。
簡単なオブジェクトならこれまでの知識で十分記述できそうですね。

#### 引数を持つフィールド

フィールドには引数を定義することができます。以下の例を見てみましょう。

```graphql
type Starship {
  id: ID!
  name: String!
  length(unit: LengthUnit = METER): Float
}
```

`length`フィールドに引数として`LengthUnit`型の`unit`が指定されていますね。
引数は必須および任意で指定することが可能です。必須としたい場合は、null 不可を示す`!`を引数の型につけましょう。任意とした場合、引数にデフォルト値を指定することも可能です。上記の例はまさしくデフォルト値を指定されていますね。
上記例だと、`Float`型の長さフィールドに、引数として`LengthUnit`の単位を、デフォルトとして`METER`(メートル)として指定しています。おそらく、スターシップの長さがメートル単位で取得できるのでしょう。

#### Query 型と Mutation 型

スキーマ定義において、Query 型と Mutation 型は特殊な意味を持ちます。この 2 つは API におけるエンドポイントを示すことができるのです。

具体的に見ていきましょう。
例えば、以下のようなクエリ文を GraphQL サーバーに送り、データが取得できたとします。

```graphql:クエリ文
query {
  hero {
    name
  }
  droid(id: "2000") {
    name
  }
}
```

```json:レスポンスデータ
{
  "data": {
    "hero": {
      "name": "R2-D2"
    },
    "droid": {
      "name": "C-3PO"
    }
  }
}
```

リクエストしたクエリ文には、`hero`および`droid`の 2 文が含まれていました。結果もそれぞれ取得できていますね。
これはすなわち、GraphQL のエンドポイントに`hero`および`droid`が用意されていることを示しています。

GraphQL におけるエンドポイントの定義は、冒頭でも触れたように Query 型もしくは Mutation 型を利用します。
今回の例で言えば、例えば以下のような型定義が必要です。

```graphql
type Query {
  hero(episode: Episode): Character
  droid(id: ID!): Droid
}
```

`Query`型に`hero`フィールドおよび`droid`フィールドが定義されていますね。ちなみに、`hero`では引数が任意となっていますが、`droid`型では必須です。
このように、Query 型に登録されたフィールドがエンドポイントとして登録されますが、型定義そのものは通常のオブジェクトと変わりません。覚えることが少なくてありがたいですね。

Query 型と Mutation 型の違いについてですが、この 2 つは行える処理が異なります。
GraphQL において、Query はデータの取得(GET)、Mutation はデータの変更(POST、PUT)を意味します。データの変更を許さず取得のみを可能とする API では、Query 型のみを定義すればいいわけですね。

#### 列挙型

みんな大好き列挙型。あまり説明の必要はないでしょう。

```graphql
enum Episode {
  NEWHOPE
  EMPIRE
  JEDI
}
```

`Episode`型として指定されたフィールドには、`NEWHOPE`、`EMPIRE`、`JEDI`のいずれかしか入らないことを示すことができます。

#### インターフェース

様々なシステムで見られるインターフェースと似通ったものだそうです。
以下のようなインターフェースがあるとします。

```graphql
interface Character {
  id: ID!
  name: String!
  friends: [Character]
  appearsIn: [Episode]!
}
```

この`Character`インターフェースを実装する型では、`id`、`name`、`friends`、`appearsIn`の 4 フィールドを実装する必要があります。
例として、Character インターフェースを実装した`Human`型と`Droid`型を見てみましょう。

```graphql
type Human implements Character {
  id: ID!
  name: String!
  friends: [Character]
  appearsIn: [Episode]!
  starships: [Starship]
  totalCredits: Int
}

type Droid implements Character {
  id: ID!
  name: String!
  friends: [Character]
  appearsIn: [Episode]!
  primaryFunction: String
}
```

Character インターフェースで定義されていたフィールド以外については自由に実装することができます。`Human`型では`starships`と`totalCredits`、`Droid`型では`primaryFunction`が実装されていますね。
同じインターフェースを実装している場合でも、型名が異なれば別個の型です。例として、以下のクエリ文と結果を見てみましょう。

```graphql:クエリ文
query HeroForEpisode($ep: Episode!) {
  hero(episode: $ep) {
    name
    primaryFunction
  }
}
```

```json:変数
{
  "ep": "JEDI"
}
```

```json:レスポンスデータ
{
  "errors": [
    {
      "message": "Cannot query field \"primaryFunction\" on type \"Character\". Did you mean to use an inline fragment on \"Droid\"?",
      "locations": [
        {
          "line": 4,
          "column": 5
        }
      ]
    }
  ]
}
```

`hero`は Character 型の値を取得するクエリです。すなわち、このクエリ文では`"ep": "JEDI"`に該当する値をもつ Character を取得しようとしているわけです。
一方で、`primaryFunction`は Droid 型にのみ実装されたフィールドであり、Character 型のすべてに含まれているとは限りません。そのため、エラーが発生しました。

取得できたデータが Droid 型の場合だったとき、一緒に`primaryFunction`の値を取得したい、といったこともあるでしょう。
その場合、詳細はクエリ文の章で紹介しますが、インラインフラグメントを使えば上手く取得することができます。

```graphql:クエリ文
query HeroForEpisode($ep: Episode!) {
  hero(episode: $ep) {
    name
    ... on Droid {
      primaryFunction
    }
  }
}
```

```json:変数
{
  "ep": "JEDI"
}
```

```json:レスポンスデータ
{
  "data": {
    "hero": {
      "name": "R2-D2",
      "primaryFunction": "Astromech"
    }
  }
}
```

#### ユニオン型

ユニオン型は TypeScript にもありますね。基本的にはあんな感じです。

```graphql
union SearchResult = Human | Droid | Starship
```

上記の場合、`SearchResult`として`Human`、`Droid`、`Starship`の 3 つを指定しています。これは`SearchResult`型を指定したフィールド等では、その実装として`Human`、`Droid`、`Starship`のいずれかが入ることを意味します。
例えば、SearchResult 型の結果を返す`search`クエリが存在する場合に、以下のような処理が行えます。

```graphql:クエリ文
{
  search(text: "an") {
    __typename
    ... on Human {
      name
      height
    }
    ... on Droid {
      name
      primaryFunction
    }
    ... on Starship {
      name
      length
    }
  }
}
```

```json:レスポンスデータ
{
  "data": {
    "search": [
      {
        "__typename": "Human",
        "name": "Han Solo",
        "height": 1.8
      },
      {
        "__typename": "Human",
        "name": "Leia Organa",
        "height": 1.5
      },
      {
        "__typename": "Starship",
        "name": "TIE Advanced x1",
        "length": 9.2
      }
    ]
  }
}
```

## 参考資料

[「GraphQL」徹底入門 ─ REST との比較、API・フロント双方の実装から学ぶ](https://employment.en-japan.com/engineerhub/entry/2018/12/26/103000)
[GraphQL のクエリを基礎から整理してみた](https://qiita.com/shunp/items/d85fc47b33e1b3a88167)
[0 から REST API について調べてみた](https://qiita.com/masato44gm/items/dffb8281536ad321fb08)
[REST を採用して気づいた「GraphQL って結局何が良いの？」について](https://www.utakata.work/entry/2019/12/02/000000)
[in between days](https://mohritaroh.hateblo.jp/entry/2019/08/18/164000)
[GraphQL 入門 - 使いたくなる GraphQL](https://qiita.com/bananaumai/items/3eb77a67102f53e8a1ad)
[GraphQL を最速でマスターするための意識改革３ヶ条](https://qiita.com/jabba/items/8d77ab86641937847673)
[Amazon が SQL++となる新しいクエリ言語、PartiQL をオープンソースで発表](https://qiita.com/Iwark/items/f9da3d1c08b7e2b6437a)
[GraphQL](https://graphql.org/)
