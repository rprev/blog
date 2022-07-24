---
title: Gatsby+GithubPagesによるブログ環境構築
date: "2022-07-25T08:10:00.000Z"
---

# はじめに
Gatsby+GithubPagesによるブログ環境構築についてまとめる。

# 構築メモ
## ツールの選定背景
静的サイトジェネレータに触れてみたかったため、https://jamstack.org/generators/ で人気のあるものから選ぶことにした。
言語的にはRubyとVue.jsの経験があるが、今回はフロントエンドということでJavaScript製のもの（Next.js、Gatsby）を優先することにした。https://gotohayato.com/content/511/  の比較記事を参考に、Gatsbyを使うこととした。

デプロイ先は、現在使用中のサービスの中で使えるということでGithubPagesを選んだ。

## 実現したい構成
1. ローカルで記事を書き、ビルドする
1. GitHubへpushする
1. ブログが公開される

なお、Gatsby公式のチュートリアルではGatsbyクラウドでビルドする方法が紹介されているが、今回はシンプルな構成にしたいので上記の通りで進める。

## 環境構築
Ubuntu on WSL2 on Windows11環境で作業する。

まずは、https://www.gatsbyjs.com/docs/tutorial/part-0/ に従ってGatsby CLIをインストールする。基本的には書いてある通りだが、curlでnvmをインストールした後に、`source ~/.bashrc`を実行した。（nvmへのパスを通すため。代わりに、WSLのシェルを落として再接続しても良い）

次に、https://www.gatsbyjs.com/docs/tutorial/part-1/ の下記手順を実行した。なお、今回GatsbyCloudは使わないため、「Build your site with Gatsby Cloud」は実行しない。

- Create a Gatsby site
- Run your site locally
- Set up a GitHub repo for your site

Gatsbyにはサイトテンプレートの機能があり、https://www.gatsbyjs.com/starters/ で公開されているものから選べる。今回はhttps://github.com/gatsbyjs/gatsby-starter-blogを使うことにしたため、`gatsby new`は、`gatsby new blog https://github.com/gatsbyjs/gatsby-starter-blog`として実行した。（blogはリポジトリ名）

ローカルのGatsbyディレクトリとGitHubのリポジトリを紐づける手順については、以下の通り行った。（`git init`を行わないと、Gatsbyディレクトリがgit管理とみなされず、その後の`git remote add`が失敗する。`git add .`と`git commit`を行わないと、ローカルにmasterブランチが存在しないため、その後の`git branch`が失敗する）

```
git init
git remote add origin https://github.com/YOUR_GITHUB_USERNAME/YOUR_GITHUB_REPO_NAME.git

git add .
git commit
git branch -M main
git push -u origin main
```

ここまでで、GitHubリポジトリのmasterブランチがmainブランチに切り替わり、mainブランチ内にソースコードが置かれるようになった。公開するのはソースコードではなくビルドした結果の方なので、そちらの設定も行っていく。

GithubPagesの公開URLは、https://アカウント名.github.io/リポジトリ名 となる。リポジトリ名部分のパスをコンフィグで設定する。リポジトリ名が「blog」の場合は以下の通り。

```
vi gatsby-config.js
-----
module.exports = {
  pathPrefix: `/blog`, ★追記
```

ソースコードはmainブランチに入れたが、公開用資材（publicフォルダ）はgh-pagesブランチに入れる。そのあたりの処理（リモートリポジトリでgh-pagesブランチを作り、publicフォルダの中身だけをpush？）をうまいことやってくれるパッケージがあるため、インストールする。
```
npm install -g gh-pages --save-dev
```

## 記事更新＆公開

content/[リポジトリ名]配下のフォルダがブログ記事になる。

`gatsby develop`で検証用ビルド&ローカルサーバ立ち上げを行える。http://localhost:8000/ へアクセスすると、編集内容を確認できるので、想定通りの表示イメージになるまで編集を続ける。

編集が済んだら、`gatsby build --prefix-paths`で本番ビルドを行う。（prefix-pathsはconfigで設定した項目）。

本番ビルドに成功したら、`gh-pages -d public`でサイトを公開する。

また、サイト公開には影響ないが、`git push origin master`でソースコードの方もpushする。

## カスタマイズ
素のままだと、https://github.com/gatsbyjs/gatsby-starter-blog の作者の情報が入っているため、以下のファイルを書き換える。

gatsby-config.js
src/components/bio.js


## GitHubでssh key使用

いつのまにかパスワード認証ができなくなっていたため、ssh鍵の作成、GitHubへの公開鍵の登録を行った。以下のコマンドはWSLの環境が再起動される度に実施する必要あり。

```
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```


## 参考
[GatsbyとGitHub Pagesで作るMarkdownブログ](https://kanamesasaki.github.io/blog/20220124-gatsby-blog/)

[GatsbyJSとTypeScriptでブログを作成して公開する(2)](https://kohsuk.tech/2020/11/25/)

[gatsby-starter-blogで作成したブログをカスタマイズする](https://crimsonality.net/gatsby/customize-practice/)
