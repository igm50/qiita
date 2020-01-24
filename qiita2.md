Docker しか知らない人のための Kubernetes 入門。

# はじめに

Kubernetes はコンテナオーケストレーションツールとして有名ですね。
AWS でも EKS としてサポートしているなど、大規模サービスにおいては欠かせない存在感のあるツールでは無いかと思います。

一方で、私のような Docker しか知らないへっぽこエンジニアには敷居が高いです。
そもそも k8s でコンテナってどこにあるの？ クラスター？ ノード？ 何それ？？？

というわけで、本記事では Docker しか知らない人のために、少しずつ k8s を理解できる記事を目指そうと思います。
※本記事では rkt については触れません。

## 3 行でわかる本記事の趣旨

- Docker しか知らない人でも k8s の概念を理解できるようになる
- コンテナという概念から少しずつ広げていき、最終的に k8s 全体を見渡せるようになる
- k8s の実装うんぬんについてはあまり触れず、概念的な構成に注視する

## Pod

![Pod図](https://d33wubrfki0l68.cloudfront.net/fe03f68d8ede9815184852ca2a4fd30325e5d15a/98064/docs/tutorials/kubernetes-basics/public/images/module_03_pods.svg)

[Pod](https://kubernetes.io/ja/docs/concepts/workloads/pods/pod-overview/) は 1 つ以上のコンテナを含む、k8s の原子単位です。
Pod には 1 つ以上のコンテナと任意の数の共有ボリュームを含むことができます。Pod は密結合なアプリケーションとリソースの集合です。
もし Pod に単一のコンテナのみが含まれている場合、その Pod はコンテナのラッパーといえます。このコンテナラッパーとしての Pod 運用は k8s で最も一般的なユースケースです。
一方で、以下の図の通りに複数のコンテナを内包する場合もあります。以下の図は、ユーザーへのリソース供給を担当する Web Server と、永続化されたリソースへのアクセスを担当する File Puller、そしてリソースの一時保存を担当する Volume を含む Pod です。

![Pod例図](https://d33wubrfki0l68.cloudfront.net/aecab1f649bc640ebef1f05581bfcc91a48038c4/728d6/images/docs/pod.svg)

Pod は基本的に使い捨てです。Pod に含まれるアプリケーションをスケールしたい場合、Pod を複数生成することで対応できます。k8s において、この Pod 複製処理はレプリケーションと呼ばれているそうです。

Pod はそれぞれ固有の IP アドレスを持ちます。すなわち、同一 Pod に複数コンテナが存在する場合、それらのコンテナは外見上同じ IP アドレスになります。

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
クラスターには必ず単一のマスターが含まれます。マスターはノードの一種ですが、通常のノードに加えて以下の要素を持ちます。

- API サーバー
  - クラスター内外間通信およびクラスター内部通信を制御する
- etcd
  - クラスター内における持続的な値を保持する分散キーバリューストア
- [コントローラマネージャー](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/)
  - クラスター内部の状態(ノード、Pod の数など)を管理する制御ループ群
- スケジューラー
  - Pod の状態を監視し、次に Pod をホストするノードを選択する

以下では、代表的なコントローラについて見ていきます。

### レプリカセット

[レプリカセット](https://kubernetes.io/ja/docs/concepts/workloads/controllers/replicaset/)は、Pod を管理するコントローラです。主に Pod の作成と削除を行います。例えば、設定した Pod 数に対し不足していれば作成を、過剰であれば削除を行います。
Pod の作成は、基本的には Pod テンプレートをもとに行われます。この Pod テンプレートはレプリカセットの設定に含まれます。そのため、複数の Pod テンプレートを管理したい場合は、複数のレプリカセットが必要になるでしょう。
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

[デプロイメント](https://kubernetes.io/ja/docs/concepts/workloads/controllers/deployment/)はレプリカセットと Pod を管理するコントローラです。

## 参考資料

[Kubernetes](https://kubernetes.io/ja/)
[Play with Kubernetes](https://labs.play-with-k8s.com/)
[イラストで学ぶ Kubernetes](https://qiita.com/baby-degu/items/ea95be49d1298b1c6a1b)
[Kubernetes: 構成コンポーネント一覧](https://qiita.com/tkusumi/items/c2a92cd52bfdb9edd613)
[Raspberry Pi でおうち Kubernetes 構築【論理編】](https://qiita.com/go_vargo/items/29f6d832ea0a289b4778)
[ブラウザ上で無料で試せる kubernetes 環境一覧](https://qiita.com/loftkun/items/7804f19c4a34f56744f6)
[Kubernetes お試しコマンドまとめ](https://qiita.com/tnagano1981/items/27f32dd3350217b94feb)
