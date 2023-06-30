---
title: 勉強会レポ：Cloudflare Meetup Tokyo Kick Off !!
tags:
  - 勉強会
  - cloudflare
  - 参加レポート
private: false
updated_at: '2023-04-26T14:18:31+09:00'
id: f94dcf46c60fd08bf5ea
organization_url_name: kddiagile
---
昨日、こちらに参加してきましたので簡易レポです。

https://cfm-cts.connpass.com/event/275461/

事例を含めいくつかセッションがありましたが、抜粋して簡単にご紹介します。


# Cloudflareとは？

エバンジェリスト亀田さんからのセッションを超雑に紹介します。

- Cloudflareを一言でいうと：「インターネットの高速化とセキュリティ」を実現するサービス群
- 世界中のデータセンターに配置されたコンポーネントで分散処理をしている
- 「すべての通信をCloudflareに通す」のが基本的な使い方。これによってDDoS防御などセキュリティ担保もできる

また、Cloudflareの設備ではIP Anycastを活用しているということです。

例えばDNS名前解決のリゾルバーなどもそうですが、複数のエッジ機器が同じIPアドレスを持っており、一つのIPアドレスへのリクエストが1:多で同時に投げられることによって、その中で最適化された経路をたどるため応答性能が上がる、といったメリットがあるようです。

https://www.cloudflare.com/ja-jp/learning/dns/what-is-anycast-dns/


# ハンズオン

実際にCloudflareで「エッジに簡易アプリケーションをデプロイして利用する」体験を行いました。以下３パートに分かれており、1時間もかからず完了できるボリュームです。

① アカウント作成

https://zenn.dev/taketakekaho/articles/5f72f4c58ab0ba

② ローカル開発環境の準備

https://zenn.dev/kameoncloud/articles/1fac9762aab4ec

③ エッジアプリのデプロイ

https://zenn.dev/kameoncloud/articles/7236a2c6ad35c0


### お絵描きでハンズオンおさらい

私は普段ゴリゴリ開発をしていないこともあり、ハンズオンは時間内に完了したものの③の後半あたりから自分が何をやっているのかよく分からなくなってしまいました。

そのため帰宅後に復習がてら絵を描いてみると理解が追いつきました。
やっぱりビジュアルで右脳に染み込ませるの、大事ですね。

#### ③の前半でやったこと（To Doアプリのデプロイ via Webエディター）
![スクリーンショット 2023-04-25 22.32.52.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/caf0633a-73ef-21b4-c3ba-2426b6cb34e2.png)

#### ③の後半でやったこと（KVデータ取得アプリのデプロイ via ローカル開発環境）
![スクリーンショット 2023-04-25 23.02.46.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/e4fd3d0f-ca5f-df67-ae79-8d2b3408e752.png)


# おわりに

いつもコマンド操作やコピペの多いハンズオンをすると「あれ、結局自分は何をやったんだっけ…？」とすぐ迷子になるタイプなので、ハンズオンの際はなるべくコピペに頼らず自分の手でコードをトレースしたり、絵を描いて全体概要を咀嚼してこそ学習効果が定着するなぁと改めて実感しました。

素敵な勉強会の企画、ありがとうございました！
