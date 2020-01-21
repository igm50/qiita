Docker しか知らない人のための Kubernetes 入門。

# はじめに

Kubernetes はコンテナオーケストレーションツールとして有名ですね。
AWS でも EKS としてサポートしているなど、大規模サービスにおいては欠かせない存在感のあるツールでは無いかと思います。

一方で、私のような Docker しか知らないへっぽこエンジニアには敷居が高いです。
そもそも k8s でコンテナってどこにあるの？ クラスター？ ノード？ 何それ？？？

というわけで、本記事では Docker しか知らない人のために、少しずつ k8s を理解できる記事を目指そうと思います。
※本記事では rkt については触れません。

## 3 行でわかる本記事の目的

- Docker しか知らない人でも k8s の概念を理解できるようになる
- コンテナという概念から少しずつ広げていき、最終的に k8s 全体を見渡せるようになる
- k8s の実装うんぬんについてはあまり触れず、概念的な構成に注視する

## Pod

![Pod図](https://d33wubrfki0l68.cloudfront.net/fe03f68d8ede9815184852ca2a4fd30325e5d15a/98064/docs/tutorials/kubernetes-basics/public/images/module_03_pods.svg)