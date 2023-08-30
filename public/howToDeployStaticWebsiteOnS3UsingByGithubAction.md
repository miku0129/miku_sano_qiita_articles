---
title: AWS S3とGitHub Actionsを使用した静的ウェブサイトの自動デプロイ
tags:
  - ['AWS S3']
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
---
# AWS S3とGitHub Actionsを使用した静的ウェブサイトの自動デプロイ

## はじめに:
これまで静的WebサイトのホスティングとしてVercelを主に使っていましたが、技術の幅を広げるためにAWS S3に挑戦しました。併せてGitHub Actionsを使用することでビルドとデプロイを自動化し開発効率を改善出来たので、自分の忘備録を兼ねて紹介します。

### この記事で紹介すること
- Reactで生成したウェブサイトを自動デプロイするためのGitHub Actionsの設定

### この記事で紹介しないこと
- AWS S3バケットのセットアップについて詳細な手順
- GitHub Actionsとの連携に必要なアクセスキーの生成と設定の詳細な手順

上記についてはすでに多くの良記事が執筆されているのでそちらを参照ください。私が参考にした記事は[こちら](https://baimamboukar.medium.com/deploy-static-websites-to-aws-s3-via-ci-cd-with-github-actions-faa8c7432a5f)です。


## 前提条件:
- AWSアカウントを持っていること
- GitHubリポジトリが準備されていること

## 手順:

### Reactアプリを生成する
```javascript
npx create-react-app my-app
```
[create-react-app](https://github.com/facebook/create-react-app)



### AWS S3バケットのセットアップ:
1. AWS Management Consoleにログインし、S3サービスに移動。
2. 新しいバケットを作成し、ウェブホスティングオプションを有効にする。
3. Access keyを生成。
注意: だいぶ端折っています。

### GitHubリポジトリの設定
mainブランチへのpushをトリガーとして、AWS S3のバケットにビルドされたファイルをデプロイするための設定について説明します。

1. GitHubリポジトリを作成する。
- リポジトリを作成し、my-appのコードをコミットする。

2. AWS S3で生成したアクセスキーをGitHub Actionsに設定する。

3. GitHub Actionsワークフローを設定する。
- ファイルの内容は以下コードをコピー＆ペーストする。

```yaml

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

3. コードに変更を加えてコミットし、ビルドされたコードがAWS S3のバケットにデプロイされることを確認する。

### 注意
- <your-bucket-name>にはあなたが作成したバケット名を入れます。また、"aws-region"の値はあなたがバケットを作成した地域に差し替えてください。
- secrets.AWS_ACCESS_KEY_ID とsecrets.AWS_SECRET_ACCESS_KEY は、GitHubリポジトリのSecretsに保存されたAWSの認証情報です。
- GitHub Actionsの設定は他にもあるので[こちら](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)を参照してください。
個人的に詰まったポイントは、buildされたファイルのみをデプロイするところです。結論として、buildとdeployを１つのsteps内に設定し、AWSへのdeployを ./build 配下で行うようにしたところ解決しました。

## まとめ:
これで、GitHub Actionsを使用してAWS S3に静的ウェブサイトを自動的にデプロイするCI/CDパイプラインが構築されました。ソースコードの変更がmainブランチへのpushをトリガーとして自動的にリリースされるため、開発効率が向上します。

### 引用
[Deploy static websites to AWS S3 via CI/CD with GitHub Actions](https://baimamboukar.medium.com/deploy-static-websites-to-aws-s3-via-ci-cd-with-github-actions-faa8c7432a5f)