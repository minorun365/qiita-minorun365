---
title: 話題のAWS CDK v2を触ってみたい！初めての環境準備からデプロイまで
tags:
  - AWS
  - 初心者
  - ハンズオン
  - CDK
  - QiitaEngineerFesta2022
private: false
updated_at: '2022-06-17T23:45:32+09:00'
id: 526eaf7d63dc51bfaa0e
organization_url_name: kddi
---
# 必要なもの

- AWSアカウント
- 操作用PC（本記事ではM1 Mac）
- CLIツール（本記事ではMac標準ターミナル）
- コードエディター（本記事ではVS Code）


# 最初の準備

### node.js ＆ npmのインストール

https://nodejs.org/en/

バージョン確認

```zsh
$ npm --version
8.11.0

$ node --version
v16.15.1
```

### AWS CDK CLIのインストール

こちらを活用

https://aws.amazon.com/jp/getting-started/guides/setup-cdk/module-two/

CDKのインストール

```zsh
$ sudo npm install -g aws-cdk
```

バージョン確認

```zsh
$ cdk --version              
2.27.0 (build 8e89048)
```


# CDKハンズオン

### ブートストラップ（AWSアカウントにS3バケットなどのCDK環境を準備）

アカウントIDを確認

```zsh
$ aws sts get-caller-identity
```

ブートストラップを実施（東京リージョンの例）

```zsh
$ cdk bootstrap aws://アカウント番号/ap-northeast-1
```


### CDKプロジェクトの作成

こちらを活用

https://aws.amazon.com/jp/getting-started/guides/setup-cdk/module-three/

PCのホーム配下にワーク用ディレクトリーを作成。

```zsh
$ mkdir ~/cdk-demo
$ cd ~/cdk-demo
```

cdk initを実行（新規プロジェクトを作成）


```zsh
$ cdk init --language typescript
```

VS Codeでワーク用ディレクトリーを開いて構造を確認
![スクリーンショット 2022-06-12 16.43.03.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/9ae8c3b9-4bb2-f6b2-a748-cdb1e9af2813.png)

`bin > cdk-demo.ts` の中身を修正。
`CdkDemoStack` 内にアカウントIDとリージョンを追記する。

```ts:cdk-demo.ts
#!/usr/bin/env node
import 'source-map-support/register';
import * as cdk from '@aws-cdk/core';
import { CdkDemoStack } from '../lib/cdk-demo-stack';

const app = new cdk.App();
new CdkDemoStack(app, 'CdkDemoStack', {
  env: { account: 'アカウントIDを記載', region: ' ap-northeast-1' },

});
```

### 初めてのデプロイ

今回はVPCを作成してみる。
CDKのEC2モジュールをインストールする。ここにVPC関連のモジュールも含まれる模様。

```zsh
$ npm install @aws-cdk/aws-ec2
```

`package.json` 内の　`dependencies` セクションにaws-ec2モジュールが追記されている。
![スクリーンショット 2022-06-12 17.14.39.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/1bdbd78e-b226-50f9-3e82-ded6237fe330.png)

`lib > cdk-demo-stack.ts` に作成したいリソースを記載していく。
![スクリーンショット 2022-06-12 17.29.13.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/08837fc1-074f-477b-e886-48d3e2cd66f6.png)

`Vpc`、`SubnetType`クラスのインポートと、`const vpc`を追加。

```ts:cdx-demo-stack.ts
import * as cdk from '@aws-cdk/core';
import { Vpc, SubnetType } from '@aws-cdk/aws-ec2';

export class CdkDemoStack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    const vpc = new Vpc(this, 'MainVpc',{
      maxAzs: 2,
      subnetConfiguration:  [
        {
          cidrMask: 24,
          name: 'public-subnet',
          subnetType: SubnetType.PUBLIC
        },
      ]
    });
  }
}
```

コードが正しいかテストしてみる。
エラーなく通れば、今回のリソースをデプロイするためのCloudFormationチェンジセットがAWS上に作成され、それを用いてtsファイルに記載したVPCリソースが構築される。

※VS Codeなどのコードエディターを使っていれば、リンター機能でコーディング中にエラーを指摘してくれる。これがCDKのメリットでもある。

```zsh
$ cdk deploy

✨  Synthesis time: 2.75s

CdkDemoStack: deploying...
CdkDemoStack: creating CloudFormation changeset...

 ✅  CdkDemoStack

✨  Deployment time: 75.48s

Stack ARN:
arn:aws:cloudformation:ap-northeast-1:********:stack/CdkDemoStack/2bf69150-ea2d-11ec-8516-0a0814765a09

✨  Total time: 78.24s
```

AWSマネジメントコンソールから実際にリソースを確認してみる。
ClourFormationスタックの「リソース」から今回作成したVPCを開くと、正しく構築されていることが確認できた。
![スクリーンショット 2022-06-12 18.00.07.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1633856/7f738f8d-2d16-e920-105c-0f10811fa84c.png)


# おわりに

VPCが作れたら、次のステップとしては好きなリソースをドキュメント参照しながらデプロイしてみると良さそう。

おかたづけは以下コマンドを実行。

```zsh
$ cdk destroy
```
