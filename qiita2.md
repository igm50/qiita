# はじめに

Kubernetes はコンテナオーケストレーションツールとして有名ですね。
AWS でも EKS としてサポートしているなど、大規模サービスにおいては欠かせない存在感のあるツールではないかと思います。

一方で、私のような Docker しか知らないへっぽこエンジニアには敷居が高いです。
そもそも Kubernetes ってコンテナどこにあるの？ クラスター？ ノード？ 何それ？？？

というわけで、本記事では Docker しか知らない人のために、少しずつ k8s を理解できる記事を目指そうと思います。
※本記事では rkt については触れません。
※本記事の画像は[Kubernetes 公式 Docs](https://kubernetes.io/ja/)から引用しています。

## 3 行でわかる本記事の趣旨

- コンテナという概念から少しずつ広げていき、最終的に Kubernetes 全体を見渡せるようになる
- Docker しか知らない人でも Kubernetes 特有の概念(Pod、レプリカセットなど)を理解できるようになる
- Kubernetes を通じたサービスの展開について理解できるようになる

なお、以下では Kubernetes を k8s と略して表記します。

## Pod

![Pod図](https://d33wubrfki0l68.cloudfront.net/fe03f68d8ede9815184852ca2a4fd30325e5d15a/98064/docs/tutorials/kubernetes-basics/public/images/module_03_pods.svg)

[Pod](https://kubernetes.io/ja/docs/concepts/workloads/pods/pod-overview/) は 1 つ以上のコンテナを含む、k8s の原子単位です。
Pod には 1 つ以上のコンテナと任意の数の共有ボリュームを含むことができます。Pod は密結合なアプリケーションとリソースの集合です。
もし Pod に単一のコンテナのみが含まれている場合(上図の Pod 1)、その Pod はコンテナのラッパーといえます。このコンテナラッパーとしての Pod 運用は、k8s で最も一般的なユースケースです。
一方で、複数のコンテナを内包する場合もあります。以下の図は、ユーザーへのリソース供給を担当する Web Server と、永続化されたリソースへのアクセスを担当する File Puller、そしてリソースの一時保存を担当する Volume を含む Pod です。

![Pod例図](https://d33wubrfki0l68.cloudfront.net/aecab1f649bc640ebef1f05581bfcc91a48038c4/728d6/images/docs/pod.svg)

Pod は基本的に使い捨てです。Pod に含まれるアプリケーションをスケールしたい場合、Pod を複数生成することで対応できます。k8s において、この Pod 複製処理はレプリケーションと呼ばれているそうです。

Pod はそれぞれ固有の IP アドレスを持ちます。すなわち、同じコンテナでも別の Pod であれば異なる IP アドレスとなります。また同一 Pod に複数コンテナが存在する場合、異なるコンテナであっても外見上同じ IP アドレスになります。

## ノード

![ノード図](https://d33wubrfki0l68.cloudfront.net/5cb72d407cbe2755e581b6de757e0d81760d5b86/a9df9/docs/tutorials/kubernetes-basics/public/images/module_03_nodes.svg)

[ノード](https://kubernetes.io/ja/docs/concepts/architecture/nodes/)は k8s におけるワーカーマシンです。いまいちピンと来ない人(私を含む)は、仮想的もしくは物理的なマシンをイメージしてください。
ノードには以下のものが含まれます。

- Pod
- コンテナランタイム
  - Docker、rkt などのこと
- [kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/)
  - k8s マスターとノード間の通信、ノード上の Pod 管理などを行う、ノードの心臓部
- [kube-proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/)
  - 各ノードにおいて、TCP、UDP、SCTP 通信を取り持つネットワークプロキシ

## クラスターとマスター

![クラスター図](https://d33wubrfki0l68.cloudfront.net/99d9808dcbf2880a996ed50d308a186b5900cec9/40b94/docs/tutorials/kubernetes-basics/public/images/module_01_cluster.svg)

k8s のアーキテクチャにおいて、クラスターはサービスにおける k8s ネットワーク全体のことを指します。
クラスターには必ず単一のマスターが含まれます。マスターは以下の要素を持ちます。

- API サーバー
  - クラスター内外間通信およびクラスター内部通信を制御する
- etcd
  - クラスター内における持続的な値を保持する分散キーバリューストア
- [コントローラマネージャー](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/)
  - クラスター内部の状態(ノード、Pod の数など)を管理する制御ループ群
- スケジューラー
  - Pod の状態を監視し、次に Pod をホストするノードを選択する

k8s において、コントローラは Pod 管理などの主要な処理を担っています。以下ではコントローラの一部を紹介したいと思います。

### レプリカセット

[レプリカセット](https://kubernetes.io/ja/docs/concepts/workloads/controllers/replicaset/)は、Pod を管理するコントローラです。主に Pod の作成と削除を行います。例えば、設定した Pod 数に対し不足していれば作成を、過剰であれば削除を行います。
Pod の作成は、基本的には Pod テンプレートをもとに行われます。Pod テンプレートに従って作成された Pod のことをレプリカと呼ぶようです(もしかしたら違うかもしれません。定義が見当たらない……)。
Pod テンプレートはレプリカセットの設定に含まれます。そのため、複数の Pod テンプレートを管理したい場合は、複数のレプリカセットが必要になるでしょう。
以下はレプリカセットの設定例です([公式 Docs](https://kubernetes.io/ja/docs/concepts/workloads/controllers/replicaset/#replicaset%e3%81%ae%e4%bd%bf%e7%94%a8%e4%be%8b)引用)。

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  # modify replicas according to your case
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
    matchExpressions:
      - { key: tier, operator: In, values: [frontend] }
  template:
    metadata:
      labels:
        app: guestbook
        tier: frontend
    spec:
      containers:
        - name: php-redis
          image: gcr.io/google_samples/gb-frontend:v3
          resources:
            requests:
              cpu: 100m
              memory: 100Mi
          env:
            - name: GET_HOSTS_FROM
              value: dns
              # If your cluster config does not include a dns service, then to
              # instead access environment variables to find service host
              # info, comment out the 'value: dns' line above, and uncomment the
              # line below.
              # value: env
          ports:
            - containerPort: 80
```

レプリカセットと Pod には所有関係があります。
レプリカセットにはセレクターという設定値があり、このセレクターと Pod のラベル情報が一致する場合、レプリカセットはその Pod を所有することが可能になります。
Pod がレプリカセットに所有されている場合、`metadata.ownerReferences`フィールドにその Pod を所有するレプリカセットに関する情報が書き込まれます。よって、`ownerReferences`フィールドが空、もしくは不正だった場合、所有者のいない Pod ということになります。
所有者がいない Pod は、即座にその Pod を所有できるレプリカセットに所有されます。
レプリカセットが削除されるとき、デフォルトではレプリカセットの所有する Pod 群は削除されます。

この通り、Pod を管理することができるレプリカセットですが、**直接レプリカセットを利用することは非推奨です**。代わりに、より上位のコントローラであるデプロイメントが利用されます。

### デプロイメント

[デプロイメント](https://kubernetes.io/ja/docs/concepts/workloads/controllers/deployment/)はレプリカセットとレプリカ(Pod)を管理するコントローラです。レプリカセットの作成や更新を通じてレプリカの数やリビジョンを管理します。
以下はデプロイメントの設定例です([公式 Docs](https://kubernetes.io/ja/docs/concepts/workloads/controllers/deployment/#creating-a-deployment)引用)。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.15.4
          ports:
            - containerPort: 80
```

Pod テンプレートが変更された場合、デプロイメントは新しい Pod テンプレートに基づいたレプリカセットを作成します。諸処の設定に基づき、新旧 2 つのレプリカセットを通じてレプリカのロールアウト(更新)を実行します。このとき、設定されたレプリカ数を下回らないようにロールアウトが行われます。
デプロイメントはロールアウトの進行状況をステータスとして持ちます。進行中、完了、失敗の 3 つで表され、`kubectl rollout status`コマンドなどで確認することが可能です。
k8s のデフォルト設定では、デプロイメントのリビジョンに応じてロールアウト履歴を保存します。最新のリビジョンで何らかの問題が発生し、レプリカの作成が正常に行えない場合は、この履歴から過去のリビジョンにロールバックすることが可能です。

### ステートフルセット

[ステートフルセット](https://kubernetes.io/ja/docs/concepts/workloads/controllers/statefulset/)はデプロイメントと Pod のスケーリングを管理する、デプロイメントより上位のコントローラです。一方で、必ずしもデプロイメントより優先して使うべきとも限りません。
ステートフルセットが適している状況として、[公式 Docs では以下の 4 項目のいずれかを要求する状況としています](https://kubernetes.io/ja/docs/concepts/workloads/controllers/statefulset/#statefulset%e3%81%ae%e4%bd%bf%e7%94%a8)。

- 安定した一意のネットワーク識別子
- 安定した永続ストレージ
- 規則的で安全なデプロイとスケーリング
- 規則的で自動化されたローリングアップデート

上記 4 項目において、「安定した」とは「Pod の更新処理によって変更されることがない」ことを指します。すなわち、「Pod の破棄によって失われるべきでないもの」を指しているかと思います。
たとえば、セッション情報をメモリなどに保持するステートフルな Web アプリケーションであれば、まさしくこの条件に一致しているかと思います。
反対に、Redis などにセッション情報を隔離しているステートレスな Web アプリケーションであれば、ステートフルセットの代わりにデプロイメントかレプリカセットを利用するべきでしょう。

ステートフルセットにおいて、Pod は全て順序とユニーク性が保証されます。
例えば N 個のレプリカを持つステートフルセットをロールアップする場合、新リビジョンの Pod は{0..N-1}の番号で作成されます。すべての新リビジョン Pod が作成されたのち、旧リビジョン Pod は{N-1..0}の番号で順番に削除されます。

ステートフルセットには複数の制約があります。詳細は[ここ](https://kubernetes.io/ja/docs/concepts/workloads/controllers/statefulset/#%e5%88%b6%e9%99%90%e4%ba%8b%e9%a0%85)をみてください。

## サービス

Pod はそれぞれ単一の IP アドレスを持ちます。
例えば、リソースへのアクセス手段を提供する Pod(以下バックエンド)が存在し、その Pod に対してクライアントのブラウザで稼働するアプリケーション(以下フロントエンド)がリクエストを送信したいとします。
このとき、フロントエンドはバックエンドへの通信経路を確立しなければなりません。一方で、バックエンドたる Pod は永続性が保証されているとは限りません。すなわち、IP アドレスが変動する可能性があります。
IP アドレスの変動するバックエンドに対し安定的な通信経路を提供する手法は数多くあると思いますが、k8s ではサービスという方法が提供されています。

![サービス図](https://d33wubrfki0l68.cloudfront.net/30f75140a581110443397192d70a4cdb37df7bfc/b5f56/docs/tutorials/kubernetes-basics/public/images/module_05_scaling2.svg)

## 参考資料

[Kubernetes](https://kubernetes.io/ja/)
[Play with Kubernetes](https://labs.play-with-k8s.com/)
[イラストで学ぶ Kubernetes](https://qiita.com/baby-degu/items/ea95be49d1298b1c6a1b)
[Kubernetes: 構成コンポーネント一覧](https://qiita.com/tkusumi/items/c2a92cd52bfdb9edd613)
[Raspberry Pi でおうち Kubernetes 構築【論理編】](https://qiita.com/go_vargo/items/29f6d832ea0a289b4778)
[ブラウザ上で無料で試せる kubernetes 環境一覧](https://qiita.com/loftkun/items/7804f19c4a34f56744f6)
[Kubernetes お試しコマンドまとめ](https://qiita.com/tnagano1981/items/27f32dd3350217b94feb)
