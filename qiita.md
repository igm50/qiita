# はじめに

最近気になっていた Apollo について、1 から調べてみました。
本記事は以下のような読者を対象にしています。

- Apollo って聞いたことはあるけど、何ができるのかよくわからない
- 気にはなってたけど、日本語の情報が少なくて調べる気になれない
- バックエンドのツールなのかフロントエンドのツールなのか、話はそれからだ

## 本記事の内容

- Apollo の簡単な概要
- Apollo を使って GraphQL サーバーを建ててみる
- Apollo を使って GraphQL サーバーを利用する React アプリを作ってみる

## Apollo の簡単な概要

公式ページは[ここ](https://www.apollographql.com/)です。
以下の画像が思想の全てを語っていると思います。

![Apollo アーキテクチャ](https://d33wubrfki0l68.cloudfront.net/c2582a3fd19bfdfc97d0144244265b2a05827467/58ebe/static/connect-easily-card-231b870a76a571c94740e1fc7c66b13a.png)

**すべてが A**(pollo)**になる。**

Apollo を利用することで、あらゆるデータが GraphQL サーバーとして集約されます。
こうすることでバックエンドの複雑性が Apollo によって隠蔽され、クライアントおよびバックエンドは Apollo サーバーとの接続にのみ注意すればよくなるわけです。
例えば REST サーバーから GraphQL サーバーに移行したいのであれば、一度 Apollo を間に挟んで GraphQL サーバーを利用できる状態にしておき、後から段階的に移行していく、などという方法がとれますね。
一方で、集約先となる GraphQL サーバーが巨大かつ複雑になることが予想されます。この問題については、[Apollo Federation](https://www.apollographql.com/docs/apollo-server/federation/introduction/)という GraphQL サービス統合機能で緩和させているようです。雰囲気だけでも知りたい方は[こちらの記事](https://blog.h-sakano.dev/entry/2019/12/11/154926)がお勧めです。

### Apollo でできること

- GraphQL サーバーが簡単に構築できる
- 各種バックエンドサービスを統合し、1 つの GraphQL エンドポイントに集約できる(本記事では触れない)
- 集約されたデータをグラフ化し、視覚的に管理することができる(本記事では触れない)
- GraphQL を利用する JavaScript クライアントアプリが簡単に作成できる

## Apollo Server を使ってみる

それでは、Apollo Server で GraphQL サーバーを作ってみましょう。
[公式の Get started](https://www.apollographql.com/docs/apollo-server/getting-started/)を参考に進めていきます。

```console
$ mkdir server
$ cd server
$ yarn init
$ yarn add apollo-server graphql
$ touch index.js
```

```javascript:index.js
const { ApolloServer, gql } = require("apollo-server");

// スキーマを定義
const typeDefs = gql`
  type Book {
    title: String
    author: String
  }

  type Query {
    books: [Book]
  }
`;

const books = [
  {
    title: "Harry Potter and the Chamber of Secrets",
    author: "J.K. Rowling"
  },
  {
    title: "Jurassic Park",
    author: "Michael Crishton"
  }
];

// booksクエリ発行時の処理を指定
const resolvers = {
  Query: {
    books: () => books
  }
};

// サーバーを起動
const server = new ApolloServer({ typeDefs, resolvers });

server.listen().then(({ url }) => {
  console.log(`🚀 Server ready at ${url}`);
});
```

node で起動してみましょう。デフォルトで 4000 番ポートを利用するので、URL は`http://localhost:4000/`になるかと思います。この URL にクエリを投げる、あるいはブラウザで直接アクセスできれば OK です。

チュートリアルそのままのコードですが、これってものすごいコードだと思いませんか?
上記から大事な部分だけを取り出してみると、これっぽっちしかありません。

```javascript
const { ApolloServer, gql } = require("apollo-server");

const typeDefs = gql`{スキーマ定義}`;

const resolvers = {
  Query: {
    // クエリに応じた関数の登録
  }
};

// サーバーを起動
const server = new ApolloServer({ typeDefs, resolvers });
server.listen().then(({ url }) => {
  // 起動時処理
});
```

これだけで GraphQL サーバーが立ち上がるんです。
しかも、クエリに登録する関数は返り値の型さえ守っていれば OK なわけです。定数を返しても裏で REST を叩いても DB 接続してもお構いなし。開発者の自由です。
本来ライブラリやフレームワークとはそういうものですが、必要十分なスマートさが美しく感じるのは私だけでしょうか。

## Apollo Client を使ってみる

GraphQL の API サーバーが用意できたので、次はクライアントアプリを作成します。Angular、Vue、Svelte など、モダンな JavaScript ライブラリは一通り用意してあるようですが、今回は React を使います。
例によって[公式の Get started](https://www.apollographql.com/docs/react/get-started/)を参考にしますが、内容はだいぶ変わります。

```console
$ mkdir client
$ cd client
$ yarn create react-app .
$ yarn add apollo-boost @apllo/react-hooks graphql
```

`src/App.js`の中身を全て消して、以下の通りに書き換えます。

```javascript:App.js
import React, { useMemo } from "react";
import ApolloClient, { gql } from "apollo-boost";
import { ApolloProvider, useQuery } from "@apollo/react-hooks";

const App = () => {
  // Apollo Clientの初期化
  const client = useMemo(
    () => new ApolloClient({ uri: "http://localhost:4000" }),
    []
  );

  return (
    <div className="App">
      <ApolloProvider client={client}>
        <div>
          <h2>
            My first Apollo app{" "}
            <span role="img" aria-label="Rocket">
              🚀
            </span>
          </h2>
          <Books />
        </div>
      </ApolloProvider>
    </div>
  );
};

const Books = () => {
  // 発行クエリの定義
  const booksQuery = useMemo(
    () => gql`
      {
        books {
          title
          author
        }
      }
    `,
    []
  );

  // クエリの発行
  const { loading, error, data } = useQuery(booksQuery);

  // 結果の表示
  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error :(</p>;

  return data.books.map(({ title, author }, index) => (
    <div key={title}>
      <h3>book{index + 1}</h3>
      <p>title:{title}</p>
      <p>author:{author}</p>
    </div>
  ));
};

export default App;
```

`ApolloProvider`は[コンテクスト](https://ja.reactjs.org/docs/context.html)を利用しているようですね。
`useQuery`は内部で[`useContext`](https://ja.reactjs.org/docs/hooks-reference.html#usecontext)を呼び出す[カスタムフック](https://ja.reactjs.org/docs/hooks-custom.html)のようです。これにより、コンポーネントのどこからでもクエリを発行できるようにしています。

では実行してみましょう。前述の Apollo Server が起動している状態で、Apollo Client を起動します。
以下のような画面が表示されるはずです。

![ApolloClient.JPG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/213374/a5e82678-7565-6d67-7341-e93ad024bd2d.jpeg)

先ほど作成した Apollo Server からデータを取得できているようですね。サクッと GraphQL クライアントアプリが作成できました。

## おわりに

というわけで、Apollo を使ってサクッとアプリを作れることがわかりました。
[Prisma](https://github.com/prisma/prisma2)や[GraphQL code generator](https://graphql-code-generator.com/)と組み合わせれば快適な GraphQL ライフを過ごせるようになりそうです。
なお、今回のコードを monorepo 形式および TypeScript でざっくり書いたものは[こちら](https://github.com/igm50/apollo-test)になります。

なお、Apollo が主眼に置いているのはデータグラフのようです。[Docs のトップ](https://www.apollographql.com/docs/)でわざわざ太字にされているくらい。
今回はサーバーとクライアントを作るだけにとどめましたが、また適当なときに触ってみたいと思います。

それでは良いお年を！

## 参考文献

[Apollo](https://www.apollographql.com/)
[Apollo Documentation Home](https://www.apollographql.com/docs/)
[TypeScript + Node.js プロジェクトのはじめかた 2019](https://qiita.com/notakaos/items/3bbd2293e2ff286d9f49#7-%E9%96%8B%E7%99%BA%E3%81%AE%E5%8A%B9%E7%8E%87%E3%82%92%E3%81%82%E3%81%92%E3%82%8B-ts-node--ts-node-dev)
[monorepo について調べた](https://qiita.com/macoshita/items/0c7bfc162e4a6db26219)
[yarn ワークスペース](https://yarnpkg.com/lang/ja/docs/workspaces/)
[GraphQL でスキーマファーストな Form Validation](https://qiita.com/seya/items/d07194509e861859a410)
[Apollo Federation のすゝめ -GraphQL とマイクロサービス-](https://blog.h-sakano.dev/entry/2019/12/11/154926)
