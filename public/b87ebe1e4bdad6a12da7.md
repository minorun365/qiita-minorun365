---
title: Kubernetes入門！主要コンポーネントを概要図で学ぼう
tags:
  - 初心者
  - インフラ
  - kubernetes
  - OSS
  - コンテナ
private: false
updated_at: '2023-06-14T00:01:54+09:00'
id: b87ebe1e4bdad6a12da7
organization_url_name: kddiagile
---
調べたらいっぱい出てくるかと思いきや、意外といいのが無かったので自分が理解しやすいように描いてみました。

# そもそもKubernetesとは？
一言でいうと「コンテナ管理ソフトウェア」です。

https://speakerdeck.com/minorun365/kontenatutehe-kubernetesru-men

オープンソースソフトウェア（OSS）なので、オンプレミスや各種パブリッククラウドなど様々なインフラストラクチャ上で利用することができます。


# 概要図
Kubernetesの主要コンポーネントは以下です。
![スクリーン ショット 2023-06-13 に 23.00.10 午後.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/fa7403a2-207e-7697-2d79-6faa5aa18b86.png)

※今回は分かりやすいよう、各コンポーネントを抽象的な役割名ではなく具体的な実装名で表現しています。

### コントロールプレーン
「管理サーバー」と捉えると分かりやすいかも知れません。
自分でサーバーを立ててセットアップすることもできますが、パブリッククラウドではマネージドサービスとして提供されています。（例：AWSのEKSなど）
- kube-apiserver：コントロールプレーンのフロントエンドとなるAPIサーバー。管理者からの操作を受け付け、各コンポーネントとやり取りする
- kube-controller-manager：「サービス」や「デプロイメント」などの各k8sオブジェクトの司令塔となるコントローラーたちを管理する
- kube-scheduler：作成されたPodとノードのアサインを決定し、etcdに書き込む
- etcd：クラスターの「あるべき状態」が記録されるキーバリューストア

### ノード（データプレーン）
実際にコンテナが動く土台サーバーの部分を指します。
- kubelet：ノード側のエージェント。自ノード内のPodが「あるべき状態」かを監視する
- kube-proxy：クラスター内外からPodへのネットワークルールを管理してiptablesを更新する
- containerd：Podを実行するためのコンテナランタイム


# ぜひ自分で触ってKubernetesを動かしてみよう！
手前味噌ですが、AWSを用いたハンズオン記事を以前に書いたのでよければハンズオンしてみてください。

https://qiita.com/minorun365/items/0441e4878f0984a9fc0a


# 参考文献
https://kubernetes.io/ja/docs/concepts/overview/components/

https://cstoku.dev/posts/2018/k8sdojo-24/

https://zenn.dev/khokohokok/articles/7b6f49ac9b2151
