<!--
title:   DockerでRailsAPIモード/Next.jsの環境構築をして、Fly.ioとVercelへデプロイしてみる
tags:    Docker,Next.js,Rails,Vercel,fly.io
id:      1163a40a86d07fa691b2
private: true
-->
## はじめに
現在、事業会社でコーダーをしながらWeb開発の勉強をしています。  
今回は、個人でWebアプリ開発を行う際にDockerを使用して、Rails（APIモード）/ Next.jsの環境構築を行い、デプロイ先にFly.ioとVercelを使用したいと考えたため、環境構築からデプロイまでの方法をまとめてみようと思います。  
## 使用技術
- Ruby 3.2.2
- Rails（APIモード） 7.0.8
- Next.js 14.0.3
- Postgresql 15.5
- Docker 24.0.6 
- Docker Compose v2.23.0
- Fly.io
- Vercel
## 全体の流れ
今回は、Webアプリ開発を想定しているので、Githubでリポジトリを作成するところから解説をしていこうと思います。
1. メインリポジトリ作成
2. フロントエンド・バックエンドディレクトリのサブモジュール化
3. Docker設定
4. フロントエンド側の環境構築（Next.js）
5. バックエンド側の環境構築（Rails API）
6. API作成
7. Next.jsでAPIを取得
8. Vercelへデプロイ
9. Fly.ioへデプロイ
10. CORS設定
### ソースコード

https://github.com/ippei-shimizu/rails-api-nextjs-verification-app


### 参考情報

https://blog.furu07yu.com/entry/rails-nextjs-monorepo-docker-setup

https://zenn.dev/taku1115/articles/6c9fa97ab37e38

