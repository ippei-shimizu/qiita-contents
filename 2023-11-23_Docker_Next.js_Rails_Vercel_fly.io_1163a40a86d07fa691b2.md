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
- `front`ディレクトと`back`ディレクトリ用のリポジトリを作成します。リポジトリ作成時に、`Add a README file
`にチェックを入れておき、リポジトリ作成時にコミットがされている状態にしておきます。
- `rails-api-nextjs-verification-app`ディレクトリ内に、`front`と`back`のサブモジュールを追加します。


```
rails-api-nextjs-verification-app $ git submodule add [フロントエンドリポジトリのSSH] front  
rails-api-nextjs-verification-app $ git submodule add [バックエンドリポジトリのSSH] back  
```

- ルートディレクトリに`.gitmodules`が作成されているかと思います。もし、作成されてなかったら、以下のコマンドで作成してください。


```
rails-api-nextjs-verification-app $ touch .gitmodules
```

```:.gitmodules
[submodule "front"]  
	path = front  
	url = [フロントエンドリポジトリのSSH]  
[submodule "back"]  
	path = back  
	url = [バックエンドリポジトリのSSH]   
```

- メインリポジトリ変更のcommitとpushを行います。


```
rails-api-nextjs-verification-app $ git add .  
rails-api-nextjs-verification-app $ git commit -m "Add: submolues"  
rails-api-nextjs-verification-app $ git push  
```

以下の画面のようになれば完了です。  

[![Image from Gyazo](https://i.gyazo.com/a22f0379af73572ca5512fe548b23c6f.png)](https://gyazo.com/a22f0379af73572ca5512fe548b23c6f)

## Docker設定
次に、Dockerの設定として`docker-compose.yml`の作成と`front` `back`ディレクトリに`Dockerfile`を作成していきます。

**ディレクト構成**

```
├── rails-api-nextjs-verification-app
    ├── front/
        ├── Dockerfile
    └── back/
        ├── Dockerfile
    ├── docker-compose.yml 
```


**docker-compose.yml**

```:docker-compose.yml
version: "3"
services:
  db:
    image: postgres:15.5
    environment:
      POSTGRES_DB: app_development
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
  back:
    build:
      context: ./back
      dockerfile: Dockerfile
    command: bash -c "rm -f tmp/pids/server.pid && bundle exec rails s -b '0.0.0.0'"
    volumes:
      - ./back:/app
    ports:
      - "3000:3000"
    depends_on:
      - db
    tty: true
    stdin_open: true
    environment:
      - RAILS_ENV=development
  front:
    build:
      context: ./front/
      dockerfile: Dockerfile
    volumes:
      - ./front:/app
    command: yarn dev -p 4000
    ports:
      - "8000:4000"
volumes:
  postgres_data:
```

<details><summary>docker-compose.ymlの各項目について</summary>

- db
  - postgres:15.5を使用しています。
  - environmentで環境変数を設定しています。
  - portsでポートマッピングを行っています。
  - volumesでデータの永続化を設定しています。
    - 通常、Dockerのコンテナ内のデータはコンテナが停止・削除されると消失します。しかし、データベースのデータは永続的に保存する必要があります。
- back
  - build → Dockerイメージのビルド方法を定義しています。
    - context → Dockerfileがあるディレクトリパスを指定してます。
    - dockerfile → Dockerfileの名前を指定します。
  - command → コンテナが起動するときに実行されるコマンドを指定します。
    - bash -c "rm -f tmp/pids/server.pid && bundle exec rails s -b '0.0.0.0'" → まず、tmp/pids/server.pidを削除して、Railsサーバーを起動しています。
  - volumes → コンテナ内のデータをホストマシンと共有するために使用されます。
  - depends_on → backが他のサービスに依存していることを指定します。
- front
  - command
    - yarn dev -p 4000 → コンテナ起動時に、ポート4000でフロントの開発サーバーを起動します。
</details>

**/front/Dockerfile**

`front`ディレクトリに`Dockerfile`を作成します。  
`Dockerfile`はイメージの設計図として機能します。必要な依存関係のインストールや、アプリケーションのコードのコピーなど、イメージを構築するために必要な情報を記載しています。

```:/front/Dockerfile
FROM node:19.4.0
WORKDIR /app
```

**/back/Dockerfile**

`back`ディレクトリに`Dockerfile`と`entrypoint.sh`を作成します。  
`entrypoint.sh`は、コンテナが開始された時に実行されるスクリプトになります。

```:/back/Dockerfile
FROM ruby:3.2.2
RUN apt-get update -qq && apt-get install -y nodejs postgresql-client

WORKDIR /app

COPY Gemfile /app/Gemfile
COPY Gemfile.lock /app/Gemfile.lock

RUN gem install bundler
RUN bundle install

COPY . /app

COPY entrypoint.sh /usr/bin/
RUN chmod +x /usr/bin/entrypoint.sh
ENTRYPOINT ["entrypoint.sh"]

EXPOSE 3002

CMD ["rails", "server", "-b", "0.0.0.0"]
```


```:entrypoint.sh
#!/bin/bash
set -e

rm -f /app/tmp/pids/server.pid

exec "$@"
```

- #!/bin/bash → Bashスクリプトであることを宣言しています。
- set -e → スクリプトが失敗したら、直ちに停止します。
- rm -f /app/tmp/pids/server.pid → Railsが生成する`server.pid`ファイルが前回のプロセスで残っているとサーバーが起動しないので、それを防ぎます。

続いて、`$ docker-compose build`を通すために、`back`ディレクトリに`Gemfile`と`Gemfile.lock`を作成します。  
`Gemfile.lock`は空のままで大丈夫です。

```:Gemfile
source "https://rubygems.org"
git_source(:github) { |repo| "https://github.com/#{repo}.git" }

ruby "3.2.2"

gem "rails", "~> 7.0.5"
```

**現在のディレクトリ構造**

```
├── rails-api-nextjs-verification-app
    ├── front/
        ├── Dockerfile
        ├── README.mb
    └── back/
        ├── Dockerfile
        ├── Gemfile
        ├── Gemfile.lock
        ├── README.mb
    ├── docker-compose.yml 
```


### 参考情報

https://blog.furu07yu.com/entry/rails-nextjs-monorepo-docker-setup

https://zenn.dev/taku1115/articles/6c9fa97ab37e38

