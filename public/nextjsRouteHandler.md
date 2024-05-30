---
title: AWS ECSでClient Componentsのみnext/serverでCookieが取得できなかった話
tags:
  - Next.js
  - React
  - TypeScript
private: true
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに
最近はフロントエンドの開発を担当しています、いっぺい（[@ippei_111](https://x.com/ippei_111)）と申します。今回は、Next.jsを使用した開発で発生した不具合とその解決方法についてまとめてみようと思います。

:::note warn
間違った説明や記載などありましたらご指摘いただけると幸いです。
:::

## 概要
Next.js の Route Handlers を使用し、ユーザーのログイン状態をbooleanで返すAPIエンドポイントを作成しています。ログイン状態の判定はCookieのアクセストークンの値を参照して行っており、ログイン時にはtrue・未ログイン時にはfalseを返すエンドポイントになっています。

今回、Cookieのアクセストークンを参照してログイン状態を返すエンドポイントは、クライアントコンポーネントでユーザー情報を表示する際に使用するものとします。

## 発生した不具合

## 環境
Next.js 13.5.5
React 18
AWS ECS

## Route Handlers

### next/server

### next/headers

## tokenの有無でログイン状態を判定するAPIエンドポイントを作成

## 本番環境でCookieを参照できない

### Server ComponentsのデータフェッチではCookieを参照できる

### Client ComponentsのデータフェッチではCookieを参照できない

### next/headersに変更したらCookieを参照できた

## 最後に
