---
title: いま話題のお嬢様翻訳アプリ用フロントエンドをAWSで構築してみましたわ〜〜！！！
tags:
  - AWS
  - S3
  - CloudFront
  - APIGateway
  - QiitaEngineerFesta2022
private: false
updated_at: '2022-12-06T00:22:03+09:00'
id: 6eb8de8de099a6f0deb7
organization_url_name: kddi
---
最近、@jiro4989さんの「テキストを壱百満天原サロメお嬢様風の口調に変換するアプリ」がバズっている。

https://github.com/jiro4989/ojosama

何なら[Web版](https://github.com/jiro4989/ojosama-web)も公開くださっていて、ワイのようなへっぽこインフラエンジニアがコントリビュート出来ることは特にないのだが、自身の勉強のためにAWSを使ってこいつにJSONを投げつけるWeb画面を構築してみた。

おそらく**AWSで独自ドメインを使ってWebサイトをホスティング**したり、**バックエンドにAPIを利用する**際に流用しやすいパターンな気がするので、構成図や構築手順を簡単に残してみる。

:::note info
本記事ならびにプログラムは[jiro4989](https://github.com/jiro4989/ojosama-web#%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%A0%E3%81%AE%E4%BD%BF%E7%94%A8%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6)様、ならびに[ANYCOLOR](https://event.nijisanji.app/guidelines/)様提示の注意事項ならびにガイドラインに則り公開しています。
:::


# 今回作ったもの

こいつ

https://cloudfront.minoruonda.com/index.html

構成図
![スクリーンショット 2022-06-19 16.16.00.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/feb6c961-b3d0-6b91-bc9d-54c1b29bcbed.png)


# つくりかた

### 1. API Gatewayのリソースを作成
[公式Readme](https://github.com/jiro4989/ojosama-web/blob/main/README.adoc)を見ると、ojosama-web APIは `Text` キーに文字列を入れたJSONを投げてやると、 `Result` キーに翻訳結果が入れられて返ってくるREST APIである模様。

```zsh
$ curl -X POST -H 'Content-Type: application/json' -d '{"Text":"これはハーブです！"}' https://ojosama.herokuapp.com/api/ojosama
{"Result":"これはおハーブですわ～！"}
```

以下の記事を参考に、POSTリクエスト用のAPIを作成してみる。

https://qiita.com/koshi_an/items/98b12216a69258358d37

AWSマネジメントコンソールより新規REST APIを作成し、POST先として　`https://ojosama.herokuapp.com/api/ojosama` を指定。
![スクリーンショット 2022-06-19 17.31.24.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/653c61da-b502-2e91-7e83-64abb689ec84.png)
試しにコンソール上からPOSTメソッドテストでリクエスト本文に `{"Text":"これはハーブです！"}` を指定してみて、 `{"Result": "これはおハーブですわ！！"}` がレスポンス本文に返ってきたら成功。

また、リソースのアクションより「CORSの有効化」を実施する。設定内容はデフォルトのままでOK。
これを `prd` ステージにデプロイしておく。


### 2. S3でHTML画面をホスティング

ここから先の流れは以下のクラメソさんブログで非常に分かりやすくまとめてくれているので、参考にさせていただいた。

https://dev.classmethod.jp/articles/cloudfront-s3-customdomain/#toc-1

まずS3コンソールから今回ホスティングに利用するバケットを作成。
CloudFront経由でOAI（オリジンアクセスアイデンティティ）を利用したアクセスを提供するため、バケット自体のパブリックアクセス設定はブロックのままでも問題なし。

また、バケットのCORS設定に以下を追加しておく。

```json
[
    {
        "AllowedHeaders": [
            "Authorization"
        ],
        "AllowedMethods": [
            "GET"
        ],
        "AllowedOrigins": [
            "https://********.execute-api.ap-northeast-1.amazonaws.com/prd" ★API GatewayのエンドポイントURLを記載する
        ],
        "ExposeHeaders": [],
        "MaxAgeSeconds": 3000
    }
]
```

このバケットに画面用のHTMLファイルを放り込む。
コードは以下の記事を参考にさせていただいた。

https://qiita.com/kidatti/items/21cc5c5154dbbb1aa27f

```html:index.html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>変換ですわ！</title>
<script type="text/javascript" src="https://ajax.googleapis.com/ajax/libs/jquery/2.1.0/jquery.min.js"></script>

<script type="text/javascript">
    $(function(){
        $("#response").html("（ここに結果が表示されます）");

        $("#button").click( function(){
            var url = "https://********.execute-api.ap-northeast-1.amazonaws.com/prd"; //API GatewayのエンドポイントURLを記載する
                var JSONdata = {
                    "Text": String($("#value1").val()) 
                };

            $.ajax({
                type : 'post',
                url : url,
                data : JSON.stringify(JSONdata),
                contentType: 'application/json',
                dataType : 'json',
                scriptCharset: 'utf-8',
                success : function(data) {
                    $("#response").html(JSON.stringify(data));
                },
                error : function(data) {

                    // Error
                    alert("error");
                    alert(JSON.stringify(data));
                    $("#response").html(JSON.stringify(data));
                }
            });
        })
    })
</script>

</head>
<body>
    <h1>お嬢様言葉に変換しますわ！<br>Webフロントエンド by みのるん</h1>
    <p>変換したい文章: <input type="text" id="value1" size="50" value="これはハーブです"></p>
    <p><button id="button" type="button">変換ですわ！</button></p>
    <textarea id="response" cols=50 rows=5 disabled></textarea>
    <br><br><br><br>
    <p>最近バズってる以下のWebアプリに<br>JSONリクエストを飛ばすためのフロントエンドです。<br></p>
    <a href="https://github.com/jiro4989/ojosama-web">https://github.com/jiro4989/ojosama-web</a>
    <br><br>
    <p>AWSのCloudFront＋S3＋API Gatewayで構築しています。</p>
    <br><br>
    <font color="gray"><p>※本プログラムは<a href="https://github.com/jiro4989">jiro4989</a>様、ならびに<a href="https://event.nijisanji.app/guidelines/">ANYCOLOR</a>様提示の<br>
    注意事項ならびにガイドラインに則り公開しています。</p></font>
</body>
</html>
```

こいつに `index.html` という名前を付けて、バケットのルートに配置しておく。
そしてバケットの静的ウェブサイトホスティング設定を有効にすると、このHTMLファイルをWebサーバー不要で外部公開してくれる。


### 3. CloudFrontでWebサイトを公開

お次はCloudFrontで上記Webサイトを配信する。
これによってS3バケットへのアクセスポイントを限定できる他、エンドユーザーの近くのエッジロケーションにコンテンツをキャッシュさせたり、独自ドメインを利用してカスタムURLからセキュアなアクセスを実現したりということが可能となる。

CloudFrontコンソールからWebディストリビューションを作成し、オリジンドメイン名にS3バケット名を指定。S3バケットへのアクセス制限を有効にしておく。
![スクリーンショット 2022-06-19 18.15.04.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/482dfafd-5c84-9221-8a74-7f8d90bff46b.png)


### 4. ACMで証明書を発行し独自ドメインURLを適用

ACM（Amazon Certificate Manager）コンソールより証明書の作成を実施。Amazon発行のパブリック証明書をリクエストする。

今回、私の場合は既にお名前.comで取得済みのドメインを利用するが、ドメインを持っていない人はRoute 53の機能でAWSコンソール内で独自ドメインを取得してしまうことも可能。これは便利。

https://dev.classmethod.jp/articles/route53-domain-2019/

ACMで発行するパブリック証明書のFQDN名は、自分のドメインにサブドメインを付けてもよい。私の場合は `cloudfront.minoruonda.com` とする。
作成者が正規のドメイン管理者であることを確認するため、確認用にDNSサーバーへCNAMEレコードを設定してみよとの課題が出される。

お名前.comのDNSレコード設定画面より、CNAMEレコードを登録。
![スクリーンショット 2022-06-19 18.29.53a.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/17a2f195-015d-e070-90cf-1e19a03d6d7e.png)
ACMが「発行済み」ステータスへ変われば完了。

その後、CloudFrontディストリビューションの代替ドメイン名へ今回設定したサブドメインを指定し、SSL証明書にもカスタム証明書を指定する。


### 動作確認

今回作成した `https://サブドメイン.独自ドメイン/index.html` へWebブラウザーからアクセスしてみる。
![スクリーンショット 2022-06-19 18.41.18.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/a16f9157-1e38-d3b1-c121-048ee33d4173.png)
フロント画面が無事に表示され、ボタンを押すと変換結果がフォーム内に表示されればOK！
（レスポンスのJSONがそのまま表示されちゃうダサい仕様なのはご愛嬌。コード改善したくなった方は本記事へのプルリクお待ちしてます！）


# おわりに

今回の構築ではCloudFront＋ACM＋S3＋API Gatewayの設定を通して、各AWSサービスの基礎知識に加えてDNSやSSL証明書、REST API、HTML/JS、CORSなどそこそこ多様なレイヤーのIT基礎知識が身につくため、意外とよい勉強材料だった。

※今回の工程の一部は、私の環境では結構前に設定済みのものを含んでいたので、実機を見つつ思い出しながら執筆しています。もし不備等あればお知らせいただけると幸いです！
