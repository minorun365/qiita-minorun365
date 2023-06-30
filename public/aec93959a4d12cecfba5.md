---
title: Semantic Kernelで「関西型言語で娘と会話する記事を書く」YametaroSkillを作ってみた
tags:
  - OpenAI
  - ゆめみ
  - ワイ記法
  - 関西型言語
  - SemanticKernel
private: false
updated_at: '2023-06-29T07:50:33+09:00'
id: aec93959a4d12cecfba5
organization_url_name: kddiagile
---
Qiita Engineer Festa、大変盛り上がっていますね！

https://qiita.com/official-campaigns/engineer-festa/2023

弊社[KDDIアジャイル開発センター](https://qiita.com/organizations/kddiagile/items)でも「これを機に社外アウトプットに挑戦しよう💪」という一大ムーブメントが社内で発生しており、大変ありがたい機会だなと思っています。

しかしQiitaといえばゆめみ社の無職やめ太郎（本名） @Yametaro  さん！
今年もFesta開幕早々から3桁いいねのバズり記事をぶっ飛ばしており流石です。弊社でも「やめ太郎さんを倒すぞー！」を合言葉に日々、アウトプットのコツをシェアしあってスキルアップを図っています。



# やめ太郎さんの人気の理由「ワイ記法」

無職やめ太郎さん（本名）といえば、7歳の娘さんと関西型言語で対話形式の技術解説を繰り広げる「ワイ記法」のQiita記事が人気です。

最近はChatGPTをはじめとする生成AIブームが凄いですが、なんと誰でもやめ太郎さん風の記事を書けるChatGPTのプロンプト芸を披露したブログが話題になっていました。

https://twitter.com/Yametaro1983/status/1673624430524923909?s=20



# 注目のLLMライブラリでYametaroSkillを実装してみよう！

OpenAIなどのLLM開発で人気のライブラリとしてはLangChainのほか、Microsoft社発のOSS「Semantic Kernel」も注目されています。

https://github.com/microsoft/semantic-kernel

Semantic Kernelには体系だった複数の機能があるのですが、そのうちの一つにプロンプトテンプレートをモジュール化できる「スキル」という機能があります。これを用いて今回、この「やめ太郎風Qiita記事」を生成できるスキルを作ってみようと思います。

https://qiita.com/nohanaga/items/430b59209b02c298ef2a

残念ながら日本語のチュートリアルは良さげなものが見当たらないのですが、公式のGetting Startedマテリアルがノートブック形式で提供されているためそちらを活用します。



# ハンズオン手順

以下のページを参照して、VS Code上でノートブックを操作できるように必要なモジュールやプラグイン類をインストールしておきます。

https://learn.microsoft.com/en-us/semantic-kernel/get-started/quick-start-guide/?tabs=Csharp

[Semantic Kernelのリポジトリ](https://github.com/microsoft/semantic-kernel)をクローンします。
`samples/notebooks` 配下にチュートリアル用のノートブックが用意されています。今回はPython版で進めます。

`00-getting-started.ipynb` を開いてノートブックの記載に従いながら、コードブロックを上から順番に実行していきます。
![スクリーン ショット 2023-06-28 に 23.27.30 午後.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/0df8d187-6f42-e3e6-2719-a35d02b0d629.png)
Step 2を実行する前に、 `.env` ファイルをノートブックと同階層に作成し、OpenAIもしくはAzure OpenAI ServiceのAPIキー等を記載しておく必要があります。

無料で手軽に試しやすいのはOpenAI公式のAPIかと思いますので、以下でアカウント作成のうえAPIキーを入手し、ノートブックのOption 1の方を実行すれば言語モデルの設定は完了です。

https://openai.com/blog/openai-api

Step 3にはサンプルコードとして、 `FunSkill` というスキルで `Joke` というファンクションを実行する例が記載されています。試しに実行してみると、`joke_function` の引数に記載したキーワードをもとにジョークを言ってくれるというアメリカンな機能が実装されています。

これを応用して、やめ太郎さん風のQiita記事を生成する `YametaroSkill` を実装してみましょう。


### YametaroSkillを作ってみる

Step 3のサンプルコードを見ると、スキルの設定が記載されたモジュールは `samples/skills` 配下に格納されていることが分かります。試しに `FunSkill/Joke` を同階層にコピーして `YametaroSkill` に作り変えていきましょう。

```zsh:semantic-kernel/samples
skills
└── YametaroSkill
    └── WaiMethod
        ├── config.json
        └── skprompt.txt
```

`config.json` は以下のように2箇所だけ修正してみます。（日本語部分）

```json:config.json
{
  "schema": 1,
  "description": "ワイ記法でブログ記事を書きます",
  "type": "completion",
  "completion": {
    "max_tokens": 1000,
    "temperature": 0.9,
    "top_p": 0.0,
    "presence_penalty": 0.0,
    "frequency_penalty": 0.0
  },
  "input": {
    "parameters": [
      {
        "name": "input",
        "description": "ブログのテーマ",
        "defaultValue": ""
      }
    ]
  }
}
```

そしてプロンプトテンプレートの実体となるのが、もう一つの `skprompt.txt` です。
こちらは前述のブログ記事を参考に、以下のようなテンプレートを記載してみます。

```skprompt.txt
〇〇について解説したワイと娘の対話文を作成してください。
ワイは40代のおっさんで〇〇に詳しくない娘の父親、娘は〇〇に詳しい7歳の娘です。
ワイは関西人で娘のことを「娘ちゃん」と呼びます。
出力形式は以下のようにお願いします。

ワイ「」
娘「」
ワイ「」
娘「」
{{$style}}
+++++

{{$input}}
+++++
```

これらを保存できたら、ノートブックに戻って `YametaroSkill` を実際に呼び出してみましょう。
先ほどのノートブックの最後にコードブロックを追加し、Step 3のサンプルを少しアレンジした以下のコードを書いてみます。

```python:Step 3
skill = kernel.import_semantic_skill_from_directory("../../skills", "YametaroSkill")
wai_method_function = skill["WaiMethod"]

print(wai_method_function("AWS"))
```

ファンクション名は `WaiMethod` と名付けました。解説するお題は「AWS」にしてみます。
書けたら実行してみましょう！



### YametoraSkill実行！

```:出力結果
AWSとはAmazon Web Servicesの略で、Amazonが提供するクラウドサービスのことです。
AWSを利用することで、サーバーの設置や運用、ネットワークの構築などを簡単に行うことができます。

ワイ「娘ちゃん、AWSって知ってる？」
娘「AWSって何？」
ワイ「AWSはAmazonが提供するクラウドサービスのことやで。サーバーの設置や運用、ネットワークの構築などを簡単に行うことができるんやで。」
娘「へー、すごいね！」
```

無事にやめ太郎さん＆娘さんを召喚できましたね 🎉
これであなたもQiitaでバズる記事が書けるかも!?
