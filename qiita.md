# はじめに

普段のお仕事では JavaJava してる筆者です。
Rust が旬らしいので、試しに簡単な GraphQL サーバーを建ててみました。
基本的な機能はライブラリに頼りましたが、Rust の文法を一通り確認するにはちょうどいいように感じました。

というわけで、Java 主体で開発を行っている筆者が、Rust の開発で感じたことをまとめます。
ちなみに最終的なコードは[こちら](https://github.com/igm50/rust-graphql-sample)。

## コンパイルエラーがすごい

慣れないうちはコンパイラにものすごい怒られます。慣れても怒られます。
けれど、コンパイラさんは優しいので丁寧にエラー原因を説明してくれます。
例えば以下の通り。

## 開発のアレコレ

ここからはオマケです。主に以下のことを書きます。

- 開発環境構築(Docker)
- 開発中に引っかかった文法

### 開発環境構築(Docker)

ローカル死んでも汚したくないマンなので、Docker を使います。ちなみに筆者は MacBook Pro をそのまま使用しています。
普通にローカルのディレクトリをマウントすると、ビルドが死ぬほど遅くなります。ビルド時のファイル同期処理がボトルネックになっているようです。
Rust のビルドディレクトリは、デフォルト設定だとルートディレクトリ直下に置かれます。[噂によると](https://qiita.com/yuki_ycino/items/cb21cf91a39ddd61f484)Docker for Mac の場合、マウントしたファイルの同期がかなり遅いんだとか。

というわけで、[こちらの記事](https://qiita.com/yagince/items/077d209ecca644398ea3)を参考に Dockerfile を作成しましょう。

```dockerfile
FROM rust:1.41-buster

WORKDIR /usr/src/app

RUN USER=root cargo init --bin

ENV CARGO_TARGET_DIR=/tmp/target

COPY ./Cargo.toml Cargo.toml
COPY ./Cargo.lock Cargo.lock

RUN cargo build \
  && rm src/main.rs

EXPOSE 8000
```

`CARGO_TARGET_DIR`でビルドディレクトリを指定しています。ファイル同期を行わないディレクトリを指定してあげましょう。これだけで(対策前に比べると)かなりサクサクになります。
他にもいろいろ書いていますが、だいたい初回ビルドを行うためのものです。初回ビルドは依存ライブラリの影響でどうしても重くなるので、先にやっておこうということですね。

### 開発中に引っかかった文法

ここでは、私が理解する(知る)までに時間がかかった文法やコツをまとめていきます。
なお、[TRPL](https://doc.rust-jp.rs/book/second-edition/)をしっかり読んでいればどれも問題ないものです。

#### サイズ不明なトレイトはスマートポインタに格納する

[TRPL のこのページ](https://doc.rust-jp.rs/book/second-edition/ch15-01-box.html)に書かれている内容です。

Rust では一部の文脈において、対象のサイズをコンパイル時に特定できている必要があります。
具体的に言うと、以下の記述は怒られます。

```rust
trait Shape {
  fn area(&self) -> f32;
}

fn print_area(s: dyn Shape) { // `s` doesn't have a size known at compile-time
  println!("{}", s.area());
}
```

`Shape`はトレイト(Java でいうインターフェースのようなもの)なので、定義した段階ではそのサイズは不明です。底辺と高さを持つ三角形かもしれませんし、半径のみを持つ円形かもしれません。
そのため、コンパイル時にはサイズが判断できず、エラーとなります。

というわけで、[Box](https://doc.rust-lang.org/std/boxed/struct.Box.html)を使いましょう。

```rust
trait Shape {
  fn area(&self) -> f32;
}

fn print_area(s: Box<dyn Shape>) {
  println!("{}", s.area());
}
```

Box に格納されたデータはヒープに配置されます。Box 自身はシンプルなスマートポインタなので、格納するデータに寄らず、サイズは常に一定です。
また、Box には`Deref`トレイトが実装されているため、普通の参照と変わらない感覚で操作することができます。

このように、サイズ不定なトレイトはスマートポインタに格納することで疑似的にサイズ特定可能になります。
Box 以外にも Rc や [Arc](https://qiita.com/qnighy/items/4bbbb20e71cf4ae527b9)等があるため、そちらもチェックしておくといいでしょう。

#### ?演算子

[TRPL のこのページ](https://doc.rust-jp.rs/book/second-edition/ch09-02-recoverable-errors-with-result.html)に書かれている内容です。

Rust には[Result 型](https://doc.rust-lang.org/std/result/enum.Result.html)があります。他の言語では Either とか呼ばれているやつですね。
失敗する可能性のある関数を定義するとき、正常なら戻り値を格納した`Ok`を、異常ならエラーなどを格納した`Err`を返却するようにしてあげる、等の使い方をします。
呼び出し側は Ok のときと Err のときの 2 通りを想定してあげなければなりません。

#### でもねでもね

こんなの最初から覚える必要なんてないんです。
ダメなところは Cargo と[RLS](https://marketplace.visualstudio.com/items?itemName=rust-lang.rust)が教えてくれるから。
サンプルコードが見たければ適当なライブラリのソースを見ればいいから。Rust の Docs ならワンクリックでソースが見れるから。
それでもわからなければ[TRPL](https://doc.rust-jp.rs/book/second-edition/)をちゃんと探せばだいたい見つかるから。
Rust の Slack コミュニティもあるらしいので、とりあえずそこに入るのが一番の早道かもしれません。エンジニアには(筆者含め)教えたがりが多いと思うので。

## 参考資料

[The Rust Programming Language](https://doc.rust-jp.rs/book/second-edition/)
[[Rust] docker コンテナにマウントしたディレクトリで build すると遅かった話し](https://qiita.com/yagince/items/077d209ecca644398ea3)
[DX を大幅に低下させる Docker for Mac を捨てて Mac 最速の Docker 環境を手に入れる](https://qiita.com/yuki_ycino/items/cb21cf91a39ddd61f484)
[Rust で DI(依存注入)する](https://qiita.com/tmtmtoo/items/5ce0166f09150d78c9ff)
[Rust の所有権まわりの基礎まとめ](https://qiita.com/koheimiya/items/f85d7fce21b37e593309)
[Rust チートシート](https://cheats.rs/)
[Rust のエラーまわりの変遷](https://qiita.com/legokichi/items/d4819f7d464c0d2ce2b8)
[Rust で Option 値や Result 値を上手に扱う](https://qiita.com/tatsuya6502/items/cd41599291e2e5f38a4a)
[Rust の?演算子](https://qiita.com/kanna/items/a0c10a0563573d5b2ed0)
[Rust の `Arc` を読む(1): Arc/Rc の基本](https://qiita.com/qnighy/items/4bbbb20e71cf4ae527b9)
[実践クリーンアーキテクチャ](https://nrslib.com/clean-architecture/)
