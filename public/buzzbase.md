---
title: 【個人開発】野球歴16年男が「野球の成績を記録して友達とランキング形式で共有できる」サービスを開発しました
tags:
  - "個人開発"
  - "Rails"
  - "Next.js"
  - "React"
  - "TypeScript"
private: false
updated_at: ""
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

## はじめに
私は小学校〜社会人2年目までの約16年間、野球漬けの毎日を過ごしていました。そんな野球を愛する男が全ての野球人に向けた「野球の成績記録・共有サービス」を開発しました。 

このサービスは、試合ごとに個人成績を記録することで、自動で「打率」や「防御率」などを計算し管理することができます。そして、LINEのグループ機能のように簡単にユーザー同士でグループを作成し、グループ内のメンバーで「打率」や「防御率」などの個人成績をランキング形式で共有・比較することができます。

作成したいと思った背景としては、私が野球部のキャプテンを務めている際に、選手の野球に対するモチベーション向上に課題を感じることが多くあり、その課題を解決するサービスを作成したいと思ったことがきっかけになります。

そんな背景があり、今回 **「BUZZ BASE | 野球の個人成績をランキング形式で共有できるアプリ」** を開発しました。

#### BUZZ BASE　https://buzzbase.jp/
![buzz-ogp.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1033689/8defd7d7-7ccd-5541-6a47-c7e8144f6654.png)

**GitHub**

【フロントエンド】

https://github.com/ippei-shimizu/buzzbase_front

【バックエンド】

https://github.com/ippei-shimizu/buzzbase_back

**ゲストユーザー情報**
野球はわかないけどサービスは触ってみたい！って方に向けてゲストユーザーを作成しました。是非、実際に使ってみてください。

【ゲストユーザー1】  
Email : buzzbase.app+1@gmail.com  
Password : password  

【ゲストユーザー2】  
Email : buzzbase.app+2@gmail.com  
Password : password     

【ゲストユーザー3】  
Email : buzzbase.app+3@gmail.com  
Password : password    

### 自己紹介
- 1996年生まれ
- 山梨県出身 / 都内の事業会社勤務
- 社会人野球2年 → 印刷会社でWeb制作1年 → 事業会社でコーダー1年 / プログラミングスクールRUNTEQ  
- 現在は、事業会社でWebサービスのコーディング・WordPressを使用したオウンドメディアの改修業務を行いながら、プログラミングスクールRUNTEQでWeb開発について学習をしています。


## サービス概要


## 使い方・機能一覧

## 使用技術

| カテゴリ | 技術 | 
| --- | --- |
| フロントエンド | Next.js 14.0.3 / React 18.2.0  / TypeScript 5.3.2 | 
| バックエンド | Ruby 3.2.2 / Ruby on Rails 7.0.8（APIモード） |
| データベース | PostgreSQL 15.5 |
| 認証 | devise toke auth 1.2.2 |
| 環境構築 | Docker |
| CI/CD | Github Actions |
| インフラ | Vercel / Heroku / Amazon S3 |
| その他 | SWR / Tailwind CSS / NextUI / Mantine / js-cookie / <br> ESLint / rubocop / CarrierWave / mini magick / letter opener web |

## インフラ構成

![infrastructure-configuration-chart.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1033689/7ad22676-a3c5-875c-7702-f6fb12a7197e.png)

## ER図

[![Image from Gyazo](https://i.gyazo.com/675a5d6a117b37be94c45cece4db3970.png)](https://gyazo.com/675a5d6a117b37be94c45cece4db3970)

## 画面遷移図

[figma - 画面遷移図](https://www.figma.com/file/zwyB9tqtr1JrFWsStPnk91/BuzzBase?type=design&node-id=0-1&mode=design)

## 技術選定理由  
野球成績記録系サービスの競合がネイティブアプリのものが多いと判断したため、ユーザーがネイティブアプリと比較して操作感でストレスを感じないよう、**シングルページアプリケーション（SPA）** を採用しました。

### フロントエンド
Reactの **「宣言的UI」** と **「コンポーネントベースのアーキテクチャ」** の特徴を活用することで、直感的にコードを書けるだけでなく、再利用性の高い設計により開発効率を向上させることが可能だと考えました。加えて、Next.jsを採用することで **ファイルベースのルーティング** や **SSG/SSR/CSR** などを使用して開発効率とパフォーマンス向上を図りました。

また、野球の成績データを多く扱う必要があったため、静的型付け言語のTypeScriptを導入することで、**型の不一致などのエラーをコンパイル時に検出** できるため、**予期せぬバグを防ぐ** ことが可能だと考えました。

UIに関しては、開発コストを抑えつつシンプルで使いやすいインターフェイスを構築したいと考えたため **NextUI** と **TailwindCSS** を採用しました。NextUIはコンポーネント数や参考にできる記事がまだ少ないですが、**デザイン性の高さ** と **TailwindCSS上に構築されているためパフォーマンスが高い** ことが今回のサービスの要件に合っていたため採用しました。

### バックエンド
Ruby on Rails を使用することで **開発速度を大幅に加速できる** と考えたためになります。そして、プログラミングスクールで学んだ言語ということもあり、Rails を採用しました。 

また、選定理由ではありませんが、**動的型付け言語の Rails** と **静的型付け言語の TypeScript** の両方を使用してみて、Rails は書くコード量が少なく「速く開発する」という特徴を改めて感じることができました。 

### インフラ・開発環境
ホスティングに Vercel を選んだ理由は、Next.js との相性が良く **静的サイト生成** や **サーバーサイドレンダリング** を簡単に利用することができる点にあります。これは、今回のサービス開発で重視した **競合サービスのネイティブアプリに劣らないユーザービリティ** を実現する上で適していると考えたからです。そして、GitHub との連携により **コードを push することで自動でデプロイ** が行われる点が、開発サイクルを加速させることができると思い採用しました。

アプリケーションサーバーに heroku を選んだ理由は、低コストで運用できてデータベースに postgreSQL を使用できる点と、インフラ構築の経験が少ない中で Rails のデプロイに関する豊富なドキュメントが利用できる点に魅力を感じたからです。

開発環境に Docker を使用した理由は、スクール内で **「Webアプリ開発のプルリクをレビューし合う会」** （参加メンバー同士でプルリクを出し合って、お互いにレビューし合いながら開発を進める）に参加したため、Docker を使用することでレビュワーの開発環境構築コストを削減したいと考えたため採用しました。

## こだわった実装

## 作成期間

## 運用して気づいた課題

## 今後の実装

## おわりに
