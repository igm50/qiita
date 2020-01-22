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

k8s のアーキテクチャにおいて、クラスターはあるサービスにおける k8s ネットワーク全体のことを指します。
クラスターには必ず単一のマスターが含まれます。マスターはノードの一種ですが、通常のノードに加えて以下の要素を持ちます。

- API サーバー
  - クラスター内外間通信およびクラスター内部通信を制御する
- etcd
  - クラスター内における持続的な値を保持する分散キーバリューストア
- [コントローラマネージャー](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/)
  - クラスター内部の状態(ノード、Pod の数など)を管理する制御ループ群
- スケジューラー
  - Pod の状態を監視し、次に Pod をホストするノードを選択する

## 参考資料

[Kubernetes](https://kubernetes.io/ja/)
[イラストで学ぶ Kubernetes](https://qiita.com/baby-degu/items/ea95be49d1298b1c6a1b)
[Kubernetes: 構成コンポーネント一覧](https://qiita.com/tkusumi/items/c2a92cd52bfdb9edd613)
