# はじめに

最近 Qiita でちらほら見るようになってきた Apollo について、1 から調べてみました。
本記事は以下のような読者を対象にしています。

- Apollo って聞いたことはあるけど、何ができるのかよくわからない
- 気にはなってたけど、日本語の情報が少なくて調べる気になれない
- バックエンドのツールなのかフロントエンドのツールなのか、話はそれからだ

## 本記事の内容

- Apollo の概要について
- Apollo を使って GraphQL サーバーを建ててみる
- Apollo を使って GraphQL サーバーを利用する React アプリを作ってみる

## Apollo の概要

### 三行まとめ

- GraphQL を基幹技術とした、JavaScript のツール群
- REST サーバーや DB サーバーなどの I/O を一本化した GraphQL サーバーを構築できる(Apollo Server)
- GraphQL サーバーを利用するクライアントアプリを簡単に作成できる(Apollo Client)

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

node で起動してみましょう。デフォルトで 4000 番ポートを利用するので、URL は`http://localhost:4000/`になるかと思います。

チュートリアルそのままのコードですが、これだけでいろんなことがわかります。
スキーマ定義もクエリ処理も JavaScript オンリーでできるのは強力ですね。ある程度慣れた人なら爆速で GraphQL サーバーが構築できそうです。
本コードのように、REST サーバーや DB を用意しなくても適当な値を返すことが可能です。
バックエンドの複雑性(REST、GraphQL、DB 等との接続)を Apollo Server で吸収し、一本化できることも大きな強みですね。

## Apollo Client を使ってみる

API サーバーが用意できたので、次はクライアントアプリを作成します。Angular、Vue、Svelte など、モダンな JavaScript ライブラリは一通り用意してあるようですが、今回は React を使います。
例によって[公式の Get started](https://www.apollographql.com/docs/react/get-started/)を参考にしますが、内容はだいぶ変わります。

```console
$ mkdir client
$ cd client
$ yarn create react-app
$ yarn add apollo-boost @apllo/react-hooks graphql
```

`App.js`の中身を全て消して、以下の通りに書き換えます。

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

  return data.books.map(({ title, author}, index }) => (
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
`useQuery`は内部で[`useContext`](https://ja.reactjs.org/docs/hooks-reference.html#usecontext)を呼び出す[カスタムフック](https://ja.reactjs.org/docs/hooks-custom.html)のようですね。これにより、コンポーネントのどこからでもクエリを発行できるようにしています。

では実行してみましょう。前述の Apollo Server が起動している状態で、Apollo Client を起動します。
以下のような画面が表示されるはずです。

![ApolloClient.JPG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/213374/a5e82678-7565-6d67-7341-e93ad024bd2d.jpeg)

先ほど作成した Apollo Server からデータを取得できているようですね。

## 参考文献

[TypeScript + Node.js プロジェクトのはじめかた 2019](https://qiita.com/notakaos/items/3bbd2293e2ff286d9f49#7-%E9%96%8B%E7%99%BA%E3%81%AE%E5%8A%B9%E7%8E%87%E3%82%92%E3%81%82%E3%81%92%E3%82%8B-ts-node--ts-node-dev)
[monorepo について調べた](https://qiita.com/macoshita/items/0c7bfc162e4a6db26219)
[yarn ワークスペース](https://yarnpkg.com/lang/ja/docs/workspaces/)
