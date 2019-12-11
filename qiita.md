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

GraphQL は、クライアント-サーバー間のデータ問い合わせに特化した言語です。
GraphQL には、データ問い合わせを行うクエリ言語とは別に、データの構造を定義するスキーマ言語をもち、このスキーマ言語によってデータ構造の型定義を行うことができます。定義された型を用いることにより、型安全なデータ通信が可能になります。

### つまり、何がどう便利になるの？

Web API の運用が簡単になるそうです。
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
誰だってリクエストを何回も投げるなんてしたくないですし、GraphQL が騒がれる理由もわかってきました。

### GraphQL でできること、できないこと

[GraphQL の仕様書](https://graphql.github.io/graphql-spec/)を一部抜粋すると、こんなことが書かれています。

```txt
GraphQL is a query language designed to build client applications by providing an intuitive and flexible syntax and system for describing their data requirements and interactions.
GraphQLは、データ要件と相互作用を記述するための直感的で柔軟な構文とシステムを提供することにより、クライアントアプリケーションを構築するために設計されたクエリ言語です。(Google翻訳)
```

```txt
GraphQL is not a programming language capable of arbitrary computation, but is instead a language used to query application servers that have capabilities defined in this specification. GraphQL does not mandate a particular programming language or storage system for application servers that implement it. Instead, application servers take their capabilities and map them to a uniform language, type system, and philosophy that GraphQL encodes. This provides a unified interface friendly to product development and a powerful platform for tool‐building.
GraphQLは、任意の計算が可能なプログラミング言語ではなく、この仕様で定義されている機能を持つアプリケーションサーバーのクエリに使用される言語です。GraphQLは、それを実装するアプリケーションサーバーに特定のプログラミング言語またはストレージシステムを強制しません。代わりに、アプリケーションサーバーはその機能を利用して、GraphQLがエンコードする統一言語、型システム、および哲学にそれらをマッピングします。これにより、製品開発に適した統合インターフェースと、ツール構築用の強力なプラットフォームが提供されます。(Google翻訳)
```

GraphQL はあくまで Web API へのデータ問い合わせを行うための言語です。すなわち、Web API を提供するサーバーの実装や、API サーバー-データベース間の処理等に関しては関与しないのです。
[GraphQL の言語仕様を利用した DB マイグレーションツール](https://www.prisma.io/)などもありますが、それらは例外と見ていいでしょう。
よって、本記事でもクライアント-サーバー間通信のみを想定した内容となります。

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

まず、`__typename`は型名を表す文字列ですね。クライアントはこの文字列をもとに、取得できたデータの型を判断できます。
次は`...on XXX`。先ほども出てきたインラインフラグメントですね。見ればそのままだと思いますが、型に応じて取得するフィールドを選択しています。
ちなみに、Human 型と Droid 型はどちらも Character 型を実装しているため、以下のように書くことも可能です。

```graphql
{
  search(text: "an") {
    __typename
    ... on Character {
      name
    }
    ... on Human {
      height
    }
    ... on Droid {
      primaryFunction
    }
    ... on Starship {
      name
      length
    }
  }
}
```

#### 入力型

入力型は、クエリの引数として用いられるフィールドの組み合わせを定義したものです。
例えば、`ReviewInput`型を以下のように定義します。

```graphql
input ReviewInput {
  stars: Int!
  commentary: String
}
```

`type`ではなく`input`を使っているのがミソです。
このとき、ReviewInput 型を用いる`CreateReviewForEpisode`クエリが存在する場合、以下のような処理が行えます。

```graphql:クエリ文
mutation CreateReviewForEpisode($ep: Episode!, $review: ReviewInput!) {
  createReview(episode: $ep, review: $review) {
    stars
    commentary
  }
}
```

```json:変数
{
  "ep": "JEDI",
  "review": {
    "stars": 5,
    "commentary": "This is a great movie!"
  }
}
```

```json:レスポンスデータ
{
  "data": {
    "createReview": {
      "stars": 5,
      "commentary": "This is a great movie!"
    }
  }
}
```

実装を確認してはいませんが、これはエピソードごとの感想を登録するためのクエリだと推測することができると思います。すなわち、ReviewInput 型は感想を登録するために必要な情報を示している、と捉えることもできますね。
このように、「ある操作を行うために必要な情報」について、入力型という形式で表現することができるわけです。
ただし、入力型は通常の型と違い、フィールドに引数を設定することはできないので注意しましょう。

### クエリ言語

ここからはクエリ言語についてです。クエリ言語を用いることにより、データの問い合わせやデータ変更依頼を行うことができます。

#### フィールド

ごくごくシンプルなクエリの例として、以下のクエリ文を見てみましょう。

```graphql:クエリ文
{
  hero {
    name
  }
}
```

```json:レスポンスデータ
{
  "data": {
    "hero": {
      "name": "R2-D2"
    }
  }
}
```

`hero`はスキーマ言語の項目で見たように、Query 型に登録されたフィールドを呼び出しています。
`name`は上記 hero の持つフィールドのうち、取得したいフィールドを指定しているわけです。結果として、`"name": "R2-D2"`が取得できています。

hero では Character 型が取得できるため、name 以外にも`appersIn`などのフィールドを持っているはずです。しかし、ここでは指定した`name`のみが返却されています。
このように、必要なフィールドのみを選択的に取得することができるため、無駄なデータでクライアントのメモリを圧迫しにくくなるわけですね。

ここで、例えば Character 型のフィールドとして`friends`を持ち、かつ`friends`はオブジェクトの配列だったとします。
その場合、以下のような処理を行うこともできます。

```graphql:クエリ文
{
  hero {
    name
    # Queries can have comments!
    friends {
      name
    }
  }
}
```

```json:レスポンスデータ
{
  "data": {
    "hero": {
      "name": "R2-D2",
      "friends": [
        {
          "name": "Luke Skywalker"
        },
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        }
      ]
    }
  }
}
```

フィールドがオブジェクトであれば、上記のように取得するフィールドを指定することができるわけです。
Query 型もオブジェクトであると考えれば、どんなクエリ文も取得するフィールドを指定しているだけと考えられますね。

#### 引数

クエリ文には引数を渡すことができます。例えば、ID が 1000 である人間を取得するクエリは以下の通りです。

```graphql:クエリ文
{
  human(id: "1000") {
    name
    height
  }
}
```

```json:レスポンスデータ
{
  "data": {
    "human": {
      "name": "Luke Skywalker",
      "height": 1.72
    }
  }
}
```

REST でデータを取得する場合、値をリクエストに含めたいときは URL もしくはクエリパラメータに入れる形になります。
一方で GraphQL の場合、ネストしたオブジェクトのフィールドそれぞれに引数を指定することができます。よって、取得するデータを柔軟に選択することができるようになるのです。
上記の例でいえば、例えば身長をメートル表記からフィート表記に変えたい場合は以下のようになります。

```graphql:クエリ文
{
  human(id: "1000") {
    name
    height(unit: FOOT)
  }
}
```

```json:レスポンスデータ
{
  "data": {
    "human": {
      "name": "Luke Skywalker",
      "height": 5.6430448
    }
  }
}
```

上記の例ですが、human に与える引数と height に与える引数では役割が異なります。
human に与える引数は取得するデータの選択、すなわち SQL でいう`WHERE`です。一方で、height の場合はデータ形式の指定をしており、おそらくこれはメートル => フィート変換処理のトリガーとなります。
このように、引数を用いた場合の振る舞いを柔軟に指定できることも、GraphQL の面白いところではないかと思います。

#### エイリアス

フィールドにはエイリアス(別名)をつけることができます。
例えば hero クエリを 2 つまとめて発行したい場合、そのままでは同じ hero なので両者の区別がつきません。
そこで、エイリアスをつけることで区別できるようになります。

```graphql:クエリ文
{
  empireHero: hero(episode: EMPIRE) {
    name
  }
  jediHero: hero(episode: JEDI) {
    name
  }
}
```

```json:レスポンスデータ
{
  "data": {
    "empireHero": {
      "name": "Luke Skywalker"
    },
    "jediHero": {
      "name": "R2-D2"
    }
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
