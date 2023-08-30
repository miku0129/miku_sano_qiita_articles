---
title: AWS S3とGitHub Actionsを使用した静的ウェブサイトの自動デプロイ
tags:
  - "AWS S3" 
  - "GitHub Actions"
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
---
# AWS S3とGitHub Actionsを使用した静的ウェブサイトの自動デプロイ

## はじめに
これまで、静的ウェブサイトのホスティングには主にVercelを使用していましたが、技術の幅を広げるためにAWS S3に挑戦しました。さらに、GitHub Actionsを導入してビルドとデプロイを自動化し、開発効率の向上を実現しました。この記事では、その手順を自分の備忘録として共有します。

### この記事で紹介する内容
- Reactで作成したウェブサイトを自動的にデプロイするためのGitHub Actionsの設定方法

### この記事では取り扱わない内容
- AWS S3バケットのセットアップに関する詳細な手順
- GitHub Actionsとの連携に必要なアクセスキーの生成と設定の詳細な手順

上記の内容については、既に他の優れた記事が多数存在するため、そちらも参考にしてみてください。私が参考にした記事は[こちら](https://baimamboukar.medium.com/deploy-static-websites-to-aws-s3-via-ci-cd-with-github-actions-faa8c7432a5f)です。

## 前提条件

以下の条件を満たしている必要があります：

- AWSアカウントを保有していること
- GitHubリポジトリが準備されていること

## 手順

### Reactアプリの生成

まず、以下のコマンドでReactアプリを生成します：

```bash
npx create-react-app my-app
```

[create-react-app](https://github.com/facebook/create-react-app)



### AWS S3バケットのセットアップ:

以下の手順でAWS S3バケットを設定します：

1. AWS Management Consoleにログインし、S3サービスに移動します。
2. 新しいバケットを作成し、ウェブホスティングオプションを有効にします。
3. アクセスキーを生成します。

注意: ここでは手順を簡略化して説明しています。

### GitHubリポジトリの設定
mainブランチへのpushをトリガーとして、AWS S3のバケットにビルドされたファイルをデプロイするための設定について説明します。

以下の手順でGitHubリポジトリを設定します：

1. リポジトリを作成し、my-appのコードをコミットします。
2. AWS S3で生成したアクセスキーをGitHub Actionsに設定します。

### GitHub Actionsワークフローの設定

以下の内容でGitHub Actionsワークフローを設定します：

```yaml:sample.yaml

name: Build and deploy static website to AWS-S3 on push

on:
  push:
    branches:
      - main
jobs: 
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Set up AWS CLI
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: eu-west-3
      
      - name: Build
        run: npm ci && npm run build

      - name: Deploy to AWS S3
        run: |
          aws s3 sync ./build s3://<your-bucket-name> --delete
```

コードに変更を加えてコミット&プッシュし、ビルドされたコードがAWS S3のバケットにデプロイされることを確認します。

### 注意

- `<your-bucket-name>`には作成したバケット名を入力します。
- `your-aws-regionには`バケットを作成した地域を指定します。
- `secrets.AWS_ACCESS_KEY_ID`と`secrets.AWS_SECRET_ACCESS_KEY`は、GitHubリポジトリのSecretsに保存されたAWSの認証情報です。
- 個人的に詰まったポイントは、buildされたファイルのみをデプロイする箇所です。結論として、buildとdeployを１つのsteps内に設定し、AWSへのdeployを ./build 配下で行うようにしたところ解決しました。

## まとめ

これで、GitHub Actionsを使用してAWS S3に静的ウェブサイトを自動的にデプロイするCI/CDパイプラインが構築されました。mainブランチへの変更がトリガーとなり、自動的にデプロイが行われるため、開発効率の向上が期待できます。

お読みいただきありがとうございました。お気づきのことがあればコメントを頂けると幸いです😊

## 引用
- [Deploy static websites to AWS S3 via CI/CD with GitHub Actions](https://baimamboukar.medium.com/deploy-static-websites-to-aws-s3-via-ci-cd-with-github-actions-faa8c7432a5f)
- [Workflow syntax for GitHub Actions](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)