<!--
title:   DockerでRailsAPIモード/Next.jsの環境構築をして、Fly.ioとVercelへデプロイしてみる
tags:    Docker,Next.js,Rails,Vercel,fly.io
id:      1163a40a86d07fa691b2
private: true
-->
## はじめに
現在、事業会社でコーダーをしながらWeb開発の勉強をしています。  
今回は、個人でWebアプリ開発を行う際にDockerを使用して、Rails（APIモード）/ Next.jsの環境構築を行い、デプロイ先にFly.ioとVercelを使用したいと考えたため、環境構築からデプロイまでの方法をまとめてみようと思います。  
もし、この記事通りに環境構築・デプロイを行っている途中で発生したエラーや、解説のミスなどありましたらコメントいただけると幸いです。
## 使用技術
- Ruby 3.2.2
- Rails（APIモード） 7.0.8
- Next.js 14.0.3
- Postgresql 15.5
- Docker 24.0.6 
- Docker Compose v2.23.0
- Node.js 19.4.0
- Fly.io
- Vercel
- Github Actions
## 全体の流れ
今回は、Webアプリ開発を想定しているので、Githubでリポジトリを作成するところから解説をしていこうと思います。
1. メインリポジトリ作成、フロントエンド・バックエンドディレクトリのサブモジュール化
2. Docker設定
3. フロントエンド側の環境構築（Next.js）
4. バックエンド側の環境構築（Rails API）
5. API作成
6. Next.jsでAPIを取得
7. Vercelへデプロイ
8. Fly.ioへデプロイ
9.  Github Actions
10. CORS設定
### ソースコード

https://github.com/ippei-shimizu/rails-api-nextjs-verification-app

## メインリポジトリ作成、フロントエンド・バックエンドディレクトリのサブモジュール化

今回は、フロントエンドとバックエンドを別々のリポジトリで管理する方法で進めたいと思います。  

<details><summary>submoduleについて</summary>

今回のディレクトリ構成を例にすると、frontディレクトリとbackディレクトリをサブモジュール化することにより、メインリポジトリからリンクはされているが、それぞれ独立したリポジトリとして扱われます。  
なので、フロントエンドとバックエンドの開発が分離されて、それぞれのリポジトリで開発を進めることができます。  

</details>

**ディレクトリ構成**

```
├── rails-api-nextjs-verification-app
  ├── front
  └── back
```

- 任意のディレクトリで、`$ mkdir [任意のディレクトリ名]`を実行します。今回は、`rails-api-nextjs-verification-app`で進めていきます。

```
$ mkdir rails-api-nextjs-verification-app
```
- `$ cd rails-api-nextjs-verification-app`で移動します。
- github上で、`rails-api-nextjs-verification-app`用のリポジトリを作成して、`git init`をします。  


```
rails-api-nextjs-verification-app $ git init 
rails-api-nextjs-verification-app $ git add README.mb
rails-api-nextjs-verification-app $ git commit -m "first commit"
rails-api-nextjs-verification-app $ git branch -M main
rails-api-nextjs-verification-app $  git remote add origin git@github.com:[ユーザーid]/[リポジトリ名].git
rails-api-nextjs-verification-app $  git push -u origin main
```
これで、リポジトリと連携ができたかと思います。
- `front`ディレクトと`back`ディレクトリ用のリポジトリを作成します。
- `rails-api-nextjs-verification-app`ディレクトリ内に、`front`と`back`のサブモジュールを追加します。


```
rails-api-nextjs-verification-app $ git submodule add [フロントエンドリポジトリのURL] front
rails-api-nextjs-verification-app $ git submodule add [バックエンドリポジトリのURL] back
```
- `.gitmodules`を作成します。


```
rails-api-nextjs-verification-app $ touch .gitmodules
```
```:.gitmodules
[submodule "front"]
	path = front
	url = [フロントエンドリポジトリのURL]
[submodule "back"]
	path = back
	url = [バックエンドリポジトリのURL] 
```
- メインリポジトリ変更のcommitとpushを行います。


```
rails-api-nextjs-verification-app $ git add .
rails-api-nextjs-verification-app $ git commit -m "Add: submolues"
rails-api-nextjs-verification-app $ git push
```
以下の画面のようになれば完了です。  

[![Image from Gyazo](https://i.gyazo.com/a22f0379af73572ca5512fe548b23c6f.png)](https://gyazo.com/a22f0379af73572ca5512fe548b23c6f)


### 参考情報

https://blog.furu07yu.com/entry/rails-nextjs-monorepo-docker-setup

https://zenn.dev/taku1115/articles/6c9fa97ab37e38

