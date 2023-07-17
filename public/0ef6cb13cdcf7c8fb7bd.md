---
title: 話題のキャッシュSaaS「Momento」をLangChainで使ってみる（チャット履歴編）
tags:
  - Python
  - OpenAI
  - ChatGPT
  - langchain
  - momento
private: false
updated_at: '2023-07-17T12:12:56+09:00'
id: 0ef6cb13cdcf7c8fb7bd
organization_url_name: kddiagile
---
:::note info
この記事は以下の続編です。特に依存性はないので好きな方からお読みください。
:::

話題のキャッシュSaaS「Momento」をLangChainで使ってみる（LLM編）
https://qiita.com/minorun365/items/ae977f32072b84f365e5


# キャッシュSaaS「Momento」がLangChainに正式対応！

AWSでDynamoDB等の開発に携わっていたKhawaja Shamsさんらが立ち上げたサーバーレスなキャッシュSaaS「Momento」。

かなりお手軽にキャッシュを利用でき、ElastiCacheやAzure Cache、Memorystore等の「マネージドだけどサーバーレスではない」キャッシュPaaSから簡単に移行できると話題になっています。

https://www.gomomento.com/jp/home-page

そんなMomentoが、大規模言語モデル（LLM）を活用したアプリケーション開発を便利にしてくれるPythonライブラリ「LangChain」のIntegrationsとして正式に組み込まれました。



# 何ができるの？

Momento側プレスリリースにも記載されている２点が直近の想定ユースケースのようです。

① LangChainからの「LLM呼び出し」のキャッシュとしてMomentoを使う
② LangChainの「チャット履歴」のセッションストアとしてMomentoを使う

https://jp.gomomento.com/blog/momento-is-now-fully-integrated-into-the-langchain-ecosystem

いずれもLangChainのドキュメントにサンプルコードが公開されているので、実際に動かしながら何が嬉しいのか試してみます。



# ハンズオンの事前準備
- 作業用ディレクトリを作成し、ルートに.envファイルを配置しAPIキーを記載
    - Momento　https://console.gomomento.com/tokens
    - OpenAI　https://platform.openai.com/account/api-keys

```zsh:.env
OPENAI_API_KEY=hoGEhoGE
MOMENTO_AUTH_TOKEN=fuGAfuGA
```

- Python用ライブラリ（OpenAI、LangChain、Momento）のインストール

```zsh:ターミナル
pip install openai langchain momento
```


# チャット履歴のセッションストアとしてMomentoを使ってみる

LangChainの公式ドキュメントはこちらです。

https://python.langchain.com/docs/modules/memory/integrations/momento_chat_message_history.html

上記はかなりさっぱりしたサンプルのみで、そもそもMomento以前に通常のMemory機能をいい感じに動かす方法を学ぶのがビギナーには結構大変です。

https://python.langchain.com/docs/modules/memory.html

とりわけ今回Momentoが対応している `ChatMessageHistory` がおまけ程度にしか紹介されておらず困っていたのですが、@kzk_maeda さんのLT資料 スライド19がすべてを解決してくれました🙏　神。

https://speakerdeck.com/kzkmaeda/deep-dive-into-momento-with-langchain?slide=19



### Momentoが、ないとき〜 🥟

```py:memory-normal.py
# レイテンシー計測開始
import time
time_start = time.time()

# 環境変数のセット
from dotenv import load_dotenv
load_dotenv()

# セッションIDを生成
import uuid
session_id = str(uuid.uuid4())

# 履歴の作成
from langchain.memory import ChatMessageHistory
history = ChatMessageHistory(session_id=session_id)

# メモリーの作成
from langchain.memory import ConversationBufferMemory
memory = ConversationBufferMemory(chat_memory=history)

# LLMの作成
from langchain.llms import OpenAI
llm = OpenAI(temperature=0.9)

# 会話の作成
from langchain.llms import OpenAI
from langchain.chains import ConversationChain
conversation = ConversationChain(llm=llm, verbose=True, memory=memory)

# AIとの会話1
input1 = "私の名前は渡辺直美です。"
conversation.predict(input=input1)

# AIとの会話2
input2 = "私の名前なんだっけ？"
conversation.predict(input=input2)

# 会話履歴を表示
print("【セッションID】")
print(session_id)
print()
print("【AIとの会話履歴】")
print(history.messages)

# レイテンシー計測終了
time_end = time.time()
tim = time_end- time_start

print()
print("【実行にかかった時間】")
print(tim)
```

実行してみると…

```zsh
【セッションID】
09b506b1-9e3c-4ccf-a675-1609f21e4df3

【AIとの会話履歴】
[HumanMessage(content='私の名前は渡辺直美です。', additional_kwargs={}, example=False), 
AIMessage(content=' こんにちは、渡辺直美さん！私の名前はAIさんです。どんなことをしたいですか？', additional_kwargs={}, example=False), 
HumanMessage(content='私の名前なんだっけ？', additional_kwargs={}, example=False), 
AIMessage(content=' あなたの名前は渡辺直美さんです！', additional_kwargs={}, example=False)]

【実行にかかった時間】
4.731361150741577
```

ちゃんと過去の会話をもとに新しい回答を生成してくれていますね。



### Momentoが、あるとき〜 🥟

```py:memory-momento.py
# レイテンシー計測開始
import time
time_start = time.time()

# 環境変数のセット
from dotenv import load_dotenv
load_dotenv()

# セッションIDを生成
import uuid
session_id = str(uuid.uuid4())

# Momentoの設定
from datetime import timedelta
cache_name = "langchain"
ttl = timedelta(days=1)

# 履歴の作成
from langchain.memory import MomentoChatMessageHistory
history = MomentoChatMessageHistory.from_client_params(session_id, cache_name, ttl)

# メモリーの作成
from langchain.memory import ConversationBufferMemory
memory = ConversationBufferMemory(chat_memory=history)

# LLMの作成
from langchain.llms import OpenAI
llm = OpenAI(temperature=0.9)

# 会話の作成
from langchain.llms import OpenAI
from langchain.chains import ConversationChain
conversation = ConversationChain(llm=llm, verbose=True, memory=memory)

# AIとの会話1
input1 = "私の名前は渡辺直美です。"
conversation.predict(input=input1)

# AIとの会話2
input2 = "私の名前なんだっけ？"
conversation.predict(input=input2)

# 会話履歴を表示
print("【セッションID】")
print(session_id)
print()
print("【AIとの会話履歴】")
print(history.messages)

# レイテンシー計測終了
time_end = time.time()
tim = time_end- time_start

print()
print("【実行にかかった時間】")
print(tim)
```

Momento版の実行結果

```zsh
【セッションID】
757a4314-7b49-4304-a19c-46a3d7a931ce

【AIとの会話履歴】
[HumanMessage(content='私の名前は渡辺直美です。', additional_kwargs={}, example=False), 
AIMessage(content=' 渡辺直美さん、よろしくお願いします。お住いはどちらですか？', additional_kwargs={}, example=False), 
HumanMessage(content='私の名前なんだっけ？', additional_kwargs={}, example=False), 
AIMessage(content=' 渡辺直美さんと申しました。何かお困りの場合は、どうぞお気軽にお申しつけください。', additional_kwargs={}, example=False)]

【実行にかかった時間】
4.419831037521362
```


### これの何が嬉しいの？

上記の実行結果を比較すると分かるように、前回のLLM編と違って実行結果が大幅に短縮されるようなものではありません。

今回Momentoが対応した `ChatMessageHistory` クラスは、人間とAIの会話履歴の管理（格納や取得）を簡単に行えるラッパーです。
実際に会話履歴を実装する際にはさらに抽象化された他の機能（`ConversationChain` など）を利用することが多そうですが、会話履歴をコンピューターのローカルメモリではなく外部ストレージに保存したい場合はこのクラスを直接利用するということです。

![スクリーン ショット 2023-07-17 に 00.11.56 午前.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/0fe1a061-2c44-d2cd-dab2-91fa976d09dd.png)

`ConversationChain` クラスのみを利用して会話履歴を実装した場合、アプリケーションを実行するサーバーやコンテナがインフラ障害等でダウンしてしまうと履歴が消失してしまいますが、 `ChatMessageHistory` クラスを用いて外部ストレージに履歴を保存しておけばアプリケーションサーバーをステートレスに保つことができるため耐障害性が向上し、オートスケーリングやサーバーレス等のモダンなインフラ機能を最大限に享受できます。



# Momentoコンソールからも確認してみる

せっかくなので今回の会話履歴をMomentoのWebコンソール上でも確認してみましょう。

https://console.gomomento.com/cache

上記のサンプルコードでは `langchain` という名前のキャッシュを定義しているため、同名のキャッシュが存在しない場合は自動で作成されています。

会話履歴ごとのセッションIDもちゃんと出力されるようにしていますが、そのまま検索してもヒットしません。
前述の前田さん資料 スライド28にも記載されていますが、デフォルトでは `message_store:` というプレフィックスが付与されるようです。

https://speakerdeck.com/kzkmaeda/deep-dive-into-momento-with-langchain?slide=28

`message_store:` + セッションID というキーでMomentoキャッシュ（ `langchain` ）を検索してみると、会話履歴の配列が確認できました！ 辞書型で格納されるようです。

![スクリーン ショット 2023-07-16 に 23.45.30 午後.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/bdaea484-beb6-4613-eb98-cfbce27aaf1a.png)



# おわりに

今回のサンプルコードはGitHubでも公開しています。

https://github.com/minorun365/langchain-momento
