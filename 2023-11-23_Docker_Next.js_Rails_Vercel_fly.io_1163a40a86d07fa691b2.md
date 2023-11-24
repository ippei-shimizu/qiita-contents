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

## リポジトリ作成とサブモジュール化

今回は、フロントエンドとバックエンドを別々のリポジトリでサブモジュール化して、メインリポジトリで読み込む方法を行います。 

<details><summary>submoduleについて</summary>

今回のディレクトリ構成を例にすると、frontディレクトリとbackディレクトリをサブモジュール化することにより、メインリポジトリからリンクはされているが、それぞれ独立したリポジトリとして扱われます。  
なので、フロントエンドとバックエンドの開発が分離されて、それぞれのリポジトリで開発を進めることができます。  

</details>

### ディレクトリ構成

```
├── rails-api-nextjs-verification-app
  ├── front
  └── back
```

- 任意のディレクトリで、`$ mkdir [任意のディレクトリ名]`を実行します。今回は、`rails-api-nextjs-verification-app`で進めていきます。

```sh
$ mkdir rails-api-nextjs-verification-app
```

- `$ cd rails-api-nextjs-verification-app`で移動します。
- github上で、`rails-api-nextjs-verification-app`用のリポジトリを作成して、`git init`をします。  


```sh
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


```sh
rails-api-nextjs-verification-app $ git submodule add [フロントエンドリポジトリのSSH] front  
rails-api-nextjs-verification-app $ git submodule add [バックエンドリポジトリのSSH] back  
```

- ルートディレクトリに`.gitmodules`が作成されているかと思います。もし、作成されてなかったら、以下のコマンドで作成してください。


```sh
rails-api-nextjs-verification-app $ touch .gitmodules
```

```sh:.gitmodules
[submodule "front"]  
	path = front  
	url = [フロントエンドリポジトリのSSH]  
[submodule "back"]  
	path = back  
	url = [バックエンドリポジトリのSSH]   
```

- メインリポジトリ変更のcommitとpushを行います。


```sh
rails-api-nextjs-verification-app $ git add .  
rails-api-nextjs-verification-app $ git commit -m "Add: submolues"  
rails-api-nextjs-verification-app $ git push  
```

以下の画面のようになれば完了です。  

[![Image from Gyazo](https://i.gyazo.com/a22f0379af73572ca5512fe548b23c6f.png)](https://gyazo.com/a22f0379af73572ca5512fe548b23c6f)

## Docker設定
次に、Dockerの設定として`docker-compose.yml`の作成と`front` `back`ディレクトリに`Dockerfile`を作成していきます。

### ディレクト構成

```
├── rails-api-nextjs-verification-app
    ├── front/
        ├── Dockerfile
    └── back/
        ├── Dockerfile
    ├── docker-compose.yml 
```


### docker-compose.yml

```yml:docker-compose.yml
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

### front/Dockerfile

`front`ディレクトリに`Dockerfile`を作成します。  
`Dockerfile`はイメージの設計図として機能します。必要な依存関係のインストールや、アプリケーションのコードのコピーなど、イメージを構築するために必要な情報を記載しています。

```Dockerfile:/front/Dockerfile
FROM node:19.4.0
WORKDIR /app
```

### back/Dockerfile

`back`ディレクトリに`Dockerfile`と`entrypoint.sh`を作成します。  
`entrypoint.sh`は、コンテナが開始された時に実行されるスクリプトになります。

```Dockerfile:/back/Dockerfile
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


```sh:entrypoint.sh
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

```Gemfile:Gemfile
source "https://rubygems.org"
git_source(:github) { |repo| "https://github.com/#{repo}.git" }

ruby "3.2.2"

gem "rails", "~> 7.0.5"
```

### 現在のディレクトリ構造

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

ルートディレクトリで`docker-compose build`を実行して、Dockerイメージをビルドします。  
このコマンドは、`docker-compose.yml`ファイルに記載された設定をもとに、Dockerイメージを作成します。

```sh
rails-api-nextjs-verification-app $ docker-compose build
```

これで、`docker-compose build`が成功すれば、DockerにImageが作成されていると思います。  
`docker images`を実行して、Imageが作成されているか確認してみてください。

## フロントエンド側の環境構築（Next.js）
次に、フロントエンド側で使用するNext.jsアプリケーションを作成していきます。  
まずは、`front`ディレクトリへ移動してください。

```sh
$ cd front
```

続いて、`$ docker-compose run --rm front yarn create next-app .`を実行します。


```sh
front $ docker-compose run --rm front yarn create next-app .
```

すると、以下のエラーが発生するかと思います。  

```sh
[##] 2/2The directory app contains files that could conflict:

  Dockerfile
  README.md

Either try using a new directory name, or remove the files listed above.

error Command failed.
Exit code: 1
Command: /usr/local/bin/create-next-app
Arguments: .
Directory: /app
Output:

info Visit https://yarnpkg.com/en/docs/cli/create for documentation about this command.
```

このエラーは、`create-next-app`コマンドが新しいNext.jsアプリケーションをセットアップする際に、実行されたディレクトリが空でないということを表しています。  
現在、frontディレクトリでは、`Dockerfile`と`README.me`が存在しているため、エラーが発生してしまいました。  

なので、一度`Dockerfile`をルートディレクトリへ移動してから、再度`$ docker-compose run --rm front yarn create next-app .`を実行します。  
`README.me`はNext.jsアプリ作成時に新規に作成されるので、ここでは削除してしまって大丈夫です。

**一時的なディレクトリ構造**

```
├── rails-api-nextjs-verification-app
    ├── front/
    └── back/
        ├── Dockerfile
        ├── Gemfile
        ├── Gemfile.lock
        ├── README.mb
    ├── docker-compose.yml 
    ├── Dockerfile  // ここに移動
```

以下のようにNext.jsアプリ作成時の設定を色々聞かれると思いますが、今回はTypeScriptとESLintとTailwindCSSとAppRouterを使用したアプリケーションを作成します。

[![Image from Gyazo](https://i.gyazo.com/682e74f322e19db459ffe4daaa3a19fa.png)](https://gyazo.com/682e74f322e19db459ffe4daaa3a19fa)

作成が成功するとfrontディレクトが以下のようになります。

[![Image from Gyazo](https://i.gyazo.com/1bd7dc28f0dd3ad0bfe77b2fca81a05f.png)](https://gyazo.com/1bd7dc28f0dd3ad0bfe77b2fca81a05f)

そうしたら、先ほど一時的に移動した`Dockerfile`をfrontディレクトリ直下に戻します。

`Dockerfile`を戻したら、frontディレクトリで`$ docker-compose up front`を実行して、Next.jsアプリケーションを起動します。  

<details><summary>docker-compose up front について</summary>

`docker-compose.yml`ファイル内に定義された`front`サービスに関連する、コンテナなどを作成します。  
</details>

そして、`http://localhost:8000/`にアクセスして、以下のようなNext.jsの初期画面が表示されたら成功です。

[![Image from Gyazo](https://i.gyazo.com/d20ca07eb84effa5c60be0ee30100cab.png)](https://gyazo.com/d20ca07eb84effa5c60be0ee30100cab)

## バックエンド側の環境構築（Rails API）
フロントエンド側のアプリケーションが作成できたら、次はバックエンド側のアプリケーションをRails APIモードで作成していきます。  

まず、`back`ディレクトリに移動します。

```sh
front $ cd ..
$ cd back
```

`back`ディレクトリで、`$ docker-compose run --rm --no-deps back bundle exec rails new . --api --database=postgresql`を実行します。  
これは、docker-composeを使用して、Railsアプリケーションを作成するためのコマンドです。  

<details><summary>docker-compose run --rm --no-deps back bundle exec rails new . --api --database=postgresql について</summary>

- docker-compose run
  - `docker-compose.yml`ファイル内のサービスを起動するために使用されます。
- --rm
  - コマンドの実行が完了した後にコンテナを自動的に削除するようにします。
- --no-deps
  - backに依存しているデータベースも一緒に起動されないようにします。
- back
  - `docker-compose.yml`ファイル内で指定されたサービス名です。
- bundle exec rails new . --api --database=postgresql
  - 新しいRailsアプリケーションをAPI専用で作成し、データベースにはPostgreSQLを使用します。
</details>

コマンドを実行すると、`README.mb`と`Gemfile`が競合を起こしてしまうので、以下の画像のように`Y`と入力して、上書き保存をします。

[![Image from Gyazo](https://i.gyazo.com/21820d060ff98de8c8b347def66b0e94.png)](https://gyazo.com/21820d060ff98de8c8b347def66b0e94)

Railsアプリケーションの作成が完了すると、`back`ディレクトリ内は以下の画像のようになります。

[![Image from Gyazo](https://i.gyazo.com/8d7e8990a0f01e3ecccec9321c3d838a.png)](https://gyazo.com/8d7e8990a0f01e3ecccec9321c3d838a)

それでは、Railsアプリケーションを起動する前に設定を行います。

#### 国際化とタイムゾーンの設定
`config/application.rb`に以下のコードを追加します。

```rb:config/application.rb
module App
  class Application < Rails::Application

    config.api_only = true
    config.time_zone = 'Tokyo'
    config.active_record.default_timezone = :local
    config.i18n.default_locale = :ja

  end
end
```

#### ホスト機能設定
`config/environments/development.rb`に`config.hosts << "api"`を追加します。  

```rb:config/environments/development.rb
Rails.application.configure do

  config.hosts << "api"
end
```

#### データベースのセットアップ
`/config/database.yml`に以下のコードを追加します。

```yml:/config/database.yml
default: &default
  adapter: postgresql
  encoding: unicode
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  username: user
  password: password
  host: db
```

次に、`back`ディレクトリで`$ docker-compose run --rm back rails db:create`を実行します。  
そうすると、以下のエラーが発生します。このエラーは、必要なRubyのgemがインストールされていないことを示します。

```sh
Could not find pg-1.5.4, puma-5.6.7, bootsnap-1.17.0, debug-1.8.0, msgpack-1.7.2, irb-1.9.1, reline-0.4.0, rdoc-6.6.0, psych-5.1.1.1, stringio-3.0.9 in locally installed gems
Run `bundle install --gemfile /app/Gemfile` to install missing gems.
```

なので、`$ docker-compose run --rm back bundle install`を実行し、`$ docker-compose build back`を実行します。  

そしたら、再度`$ docker-compose run --rm back rails db:create`を実行します。 
データベースのセットアップが完了したはずなので、`$ docker-compose up`を実行してアプリケーションを起動します。  

`http://0.0.0.0:3000/`にアクセスして、Railsの初期画面が表示されたらRailsアプリケーションの作成は完了です。

[![Image from Gyazo](https://i.gyazo.com/3ecb22d1dec2e9101897c20f3ca15754.png)](https://gyazo.com/3ecb22d1dec2e9101897c20f3ca15754)


ここまでが、DockerでRailsAPIとNext.jsの環境構築が完了になります。  
この後は、実際にRailsで簡単なAPIを作成してみて、Next.jsで非同期処理を行いたいと思います。  

## API作成
次に、Railsアプリケーションで`scaffold`を使用して、簡単なAPIを作成してみたいと思います。  
また、Next.js側で作成したAPIと簡単なやり取りができるところまで実装してみます。  

### scaffold追加
`back`ディレクトリで、`docker-compose run --rm back bundle exec rails g scaffold post title:string`を実行します。

```sh
back $ docker-compose run --rm back bundle exec rails g scaffold post title:string
```

次に、`docker-compose run --rm back bundle exec rails db:migrate`を実行します。

```sh
back $ docker-compose run --rm back bundle exec rails db:migrate
```

次に、`seeds.rb`でテストデータを作成します。  

```rb:db/seeds.rb
Post.create!(
  [
    { title: '野球のルール基礎知識' },
    { title: 'プロ野球選手のトレーニング方法' },
    { title: '野球の歴史とは' },
    { title: 'メジャーリーグと日本プロ野球の違い' },
    { title: '野球用具の選び方' },
    { title: '野球のポジション紹介' },
    { title: '野球の戦術入門' },
    { title: '子供向け野球教室の選び方' },
    { title: '高校野球の魅力' },
    { title: '野球観戦の楽しみ方' },
    { title: '野球のスコアブックのつけ方' },
    { title: '野球の審判の役割' },
    { title: '野球におけるピッチングの技術' },
    { title: 'バッティングの基本' },
    { title: '野球の名言集' },
    { title: '野球のトレーニング用品紹介' },
    { title: '野球選手の食事管理' },
    { title: '野球の怪我の予防と対処法' },
    { title: '野球の上達法' },
    { title: '野球の国際大会について' }
  ]
)
```

`back $ docker-compose run --rm back bundle exec rails db:seed`を実行して、テストデータを作成します。 
この後に、`http://0.0.0.0:3000/posts`にアクセスすると、JSON形式で先ほど作成したテストデータが表示されているかと思います。  

### rack-cors追加
次に、`gem "rack-cors"`を追加して、CORSを管理する設定を行います。  
CORSとは、セキュリティの観点から、ブラウザから異なるオリジン（ドメイン・プロトコル・ポート）からのスクリプトによるリソースの読み込みを制限するものです。  
なので、RailsAPIをフロントからAPIリクエストを行った時に、Rails側でCORS設定を行っていないオリジンだった場合、エラーになってしまいます。  

`Gemfile`に`gem "rack-cors"`がコメントアウトされていると思うので、コメントアウトして、`config/initializers/cors.rb`を以下のように変更します。 

```rb:config/initializers/cors.rb
Rails.application.config.middleware.insert_before 0, Rack::Cors do
  allow do
    origins 'localhost:8000', '127.0.0.1:8000'

    resource "*",
      headers: :any,
      methods: [:get, :post, :put, :patch, :delete, :options, :head]
  end
end
```

こうすることで、`localhost:8000`と`127.0.0.1:8000`からのAPIリクエストを許可することができます。  
そしたら、`back`ディレクトリで`$ docker-compose run --rm back bundle install`を実行します。

```sh
back $ docker-compose run --rm back bundle install
```

そして、再ビルドします。

```sh
back $ docker-compose build back
```

### Next.jsの実装
次に、Next.js側で「記事のタイトルのみ投稿するフォーム」と「記事一覧を取得」する実装を行ってみたいと思います。  
以下がTypeScriptを使用したコードになります。（不要なcssは削除しました）  

<details><summary>page.tsx</summary>


```tsx:app/page.tsx
"use client";
import React, { useEffect, useState } from "react";

type Post = {
  id: number;
  title: string;
};

export default function Home() {
  const [posts, setPosts] = useState<Post[]>([]);
  const [newTitle, setNewTitle] = useState("");

  const fetchPosts = async () => {
    try {
      const response = await fetch(`${process.env.NEXT_PUBLIC_API_URL}/posts`);
      if (!response.ok) {
        throw new Error("データの取得に失敗しました");
      }
      const data = await response.json();
      setPosts(data);
    } catch (error) {
      console.error(error);
    }
  };

  useEffect(() => {
    fetchPosts();
  }, []);

  const handleSubmit = async (event: React.FormEvent) => {
    event.preventDefault();
    try {
      const response = await fetch(`${process.env.NEXT_PUBLIC_API_URL}/posts`, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ title: newTitle }),
      });
      if (!response.ok) {
        throw new Error("投稿に失敗しました");
      }
      setNewTitle("");
      fetchPosts();
    } catch (error) {
      console.error(error);
    }
  };

  return (
    <main className="flex min-h-screen flex-col items-center justify-center p-24">
      <h2 className="text-3xl mb-4">記事の一覧</h2>
      <form onSubmit={handleSubmit} className="mt-4 mb-4">
        <input
          type="text"
          value={newTitle}
          onChange={(e) => setNewTitle(e.target.value)}
          placeholder="新しい投稿のタイトル"
          className="mr-2 p-2 border"
        />
        <button type="submit" className="p-2 bg-blue-500 text-white">
          投稿する
        </button>
      </form>
      <ul>
        {posts.map((post) => (
          <li key={post.id}>{post.title}</li>
        ))}
      </ul>
    </main>
  );
}
```

</details>

[![Image from Gyazo](https://i.gyazo.com/dee2f5b7df19fbd29e5f12a702adb80b.png)](https://gyazo.com/dee2f5b7df19fbd29e5f12a702adb80b)




## 参考情報

https://blog.furu07yu.com/entry/rails-nextjs-monorepo-docker-setup

https://zenn.dev/taku1115/articles/6c9fa97ab37e38

