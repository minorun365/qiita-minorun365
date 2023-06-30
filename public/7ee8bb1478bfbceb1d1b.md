---
title: AWS CLIのプロファイルで複数アカウントを使い分けよう！
tags:
  - AWS
  - 初心者
  - aws-cli
private: false
updated_at: '2023-06-16T09:27:44+09:00'
id: 7ee8bb1478bfbceb1d1b
organization_url_name: kddiagile
---
色んなAWSアカウントや複数のPCを使い分けていると、AWS CLI操作をするたびに「えっと…どのAWSアカウント向けに設定してたっけ？」ってなっちゃうので、自分用の備忘をかねてまとめました。


# 環境前提
- M1以降のMac
- ターミナルはzshを利用


# 現状確認
- そもそもこのPC、AWS CLI入ってたっけ？

```zsh:ターミナル
aws --version
```

- 今このPCに登録されているIAMアクセスキーの一覧は？

```zsh:ターミナル
cat ~/.aws/credentials
```

- 今このPCはどのプロファイルを向いている？

```zsh:ターミナル
aws configure list
```

- 現在のプロファイルへ試しに実行しやすい無難なコマンド

```zsh:ターミナル
aws s3 ls
```

※当該アカウントのS3バケット一覧を表示してくれます。


# AWS CLIのインストール
入ってなかったら入れましょう。

- GUIインストーラー（Mac用）

https://awscli.amazonaws.com/AWSCLIV2.pkg

※「すべてのユーザー用」にインストールするとパス設定が不要になるので楽です。

- インストールされたことの確認

```zsh:ターミナル
aws --version
```


# AWS CLIの構成ファイルは2つ！

ホームディレクトリ配下の隠しディレクトリ `~/.aws` に以下の2ファイルがあります。

### credentials（認証情報ファイル）
ログインに用いるIAMユーザーのアクセスキー情報を記載します。
複数のIAMユーザー情報を列挙して切り替えることができます。これらをプロファイルと言います。

```ini:~/.aws/credentials
[default]
aws_access_key_id=AKIAIOSFODNN7EXAMPLE
aws_secret_access_key=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY

[project1-dev]
aws_access_key_id=AKIAI44QH8DHBEXAMPLE
aws_secret_access_key=je7MtGbClwBF/2Zp9Utk/h3yCo8nvbEXAMPLEKEY

[project1-prd]
aws_access_key_id=AKIAI44QA8DHBEXAMPLE
aws_secret_access_key=je7MtGbClwBF/2Zp9Utk/h3yCo8avbEXAMPLEKEY
```


### config（設定ファイル）
前述のcredentialsファイルで定義した各プロファイルについて、デフォルトで利用するAWSリージョンなどを定義できるファイルです。

```ini:~/.aws/config
[default]
region=ap-northeast-1

[profile project1-dev]
region=ap-northeast-3

[profile project1-prd]
region=ap-northeast-3
```


# プロファイル切り替え方法
AWS CLI実行時に利用されるプロファイルは、Macの環境変数 `AWS_PROFILE` で指定できます。

- 現在このPCで有効なプロファイルを確認する

```zsh:ターミナル
echo $AWS_PROFILE
```

※出力なしの場合はデフォルトプロファイルが利用ます。

- 環境変数のセット

```zsh:ターミナル
export AWS_PROFILE=project1-dev
```

Mac（UNIX系のOS）では環境変数はターミナルを閉じるたびにクリアされてしまうので、ターミナル再起動後は環境変数のセットを忘れない様にしましょう。


# 注意点
今回紹介した認証方法は「Long-term credentials」と呼ばれますが、長期の認証情報が特定のデバイス内に残ってしまうことからセキュリティリスクの観点で非推奨とされています。

https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/cli-authentication-user.html

可能なかぎりアクセスキーの発行は避け、ユースケースに応じた別の手段を用いて目的達成するようにしましょう。
また、やむなく発行したアクセスキーは定期的にローテーションしてセキュリティリスクを最小化しましょう。


# 参考文献

- AWS CLIユーザーガイド

https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/cli-chap-welcome.html

- AWS CLIのプロファイル切り替えをいい感じにする（@crossroad0201 さま）

https://qiita.com/crossroad0201/items/f84b3e2fece41750755b
