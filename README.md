# Saddlerで悪いロボット

## はじめに

コードレビューにおいて、指摘するというは勇気とエネルギーがいるものです。特に新しい言語での開発プロジェクトは、足りないものだらけなので自動化や省力化はとても重要な要素になります。
LEGOには、[悪いロボット](https://www.lego.com/ja-jp/minifigures/characters/evil-robot-23fc8e61ce564d138e9a5d4d90e0254e)というキャラクターがいます。人間と正反対のことをするNANDなロボットです。
今回は、開発プロジェクトの悪いロボット、コードレビューの際に指摘事項をコメントしてくれるbotについて、コードを交えながら紹介します。

## 用意するもの

コードベースは、JavaScript(ES2015)とします。

[mediba-Kitada/evilbot-js: Saddlerで悪いロボット](https://github.com/mediba-Kitada/evilbot-js)


### [GitHub](https://github.com)のアカウント

コードベースの管理は、GitHubで行うものとします。

### [TravisCI](https://travis-ci.com/)のアカウント

CIは、TravisCIで行うものとします。
[travis](https://github.com/travis-ci/travis.rb) gemを使うとCLIで操作出来ます。

### [Saddler](http://packsaddle.org/)

SaddlerをCI環境で稼働させ、リント結果を指摘事項としてコメントにパイプさせます。

### [yarn](https://yarnpkg.com/)

モジュール管理には、yarnを用います。

### [ESLint](http://eslint.org/)

リンターには、ESLintを用いいます。

### [standardJS](http://standardjs.com/)

コード規約には、standardJSを用いいます。
ESLintのプラグインとして設定して、リントします。

## 手順

### Node.jsのバージョン設定

実際のプロジェクトでは、AWS Lambdaを実行環境としてますので、4.3.2を指定しておきます。
また、Node.js v4以上の場合、TravisCIでyarnが利用出来るので、```.node-version```ファイルを配置して、バージョンを指定しておきます。

### モジュール管理

yarnかわいいやーん。

```
# 初期化
% yarn init
yarn init v0.20.3
question name (evilbot-js):
question version (1.0.0):
question description: Evil Bot in JavaScript
question entry point (index.js):
question repository url (git@mediba-github:mediba-kitada/evilbot-js.git):
question author: mediba-kitada <kitada@mediba.jp>
question license (MIT): none
success Saved package.json
✨  Done in 101.25s.

# 各モジュールの導入
% yarn add eslint standard eslint-config-standard
yarn add v0.20.3
warning evilbot-js@1.0.0: License should be a valid SPDX license expression
[1/4] 🔍  Resolving packages...
[2/4] 🚚  Fetching packages...
[3/4] 🔗  Linking dependencies...
中略
warning evilbot-js@1.0.0: License should be a valid SPDX license expression
✨  Done in 23.82s.
```

travisとSaddlerもGemfileで管理します。
Rubyのバージョン指定は必須ではありませんが、TravisCIの環境(2.2.5)に合わせておきます。

```
# Rubyのバージョン指定
% rbenv install 2.2.5
ruby-build: use openssl from homebrew

% vi .ruby-version
2.2.5

# bundlerのインストール
% gem install bundle

# 各モジュールの導入
% bundle init
% vi Gemfile

group :local do
  gem 'travis'
end

group :ci do
  gem 'checkstyle_filter-git' 
  gem 'saddler' 
  gem 'saddler-reporter-github'
end

# ローカルではtravisだけ必要となる
% bundle install --without ci --path ./vendor/bundle
```

### リンターの設定

ESLintをリンターとし、standardJSをプラグインとして設定します。

```
# ESLintの設定ファイルを生成、編集
% vi .eslintrc
{
  "extends": "standard"
}

# npm scriptsに登録
# フォーマットは、checkstyleを指定しておく
% vi package.json
"scripts": {
  "lint": "eslint -f checkstyle"
},

# コマンド例
% yarn lint index.js
yarn lint v0.20.3
$ eslint -f checkstyle index.js
<?xml version="1.0" encoding="utf-8"?><checkstyle version="4.3"><file name="/Users/kitada/project/evilbot-js/evilbot-js/index.js"></file></checkstyle>
✨  Done in 1.14s.
```

### JavaScriptアプリケーションを用意する

今回は、チームのバイブルとなっている[はじめてのJavaScript]() 14章からサンプル用のアプリケーションを拝借します。
masterブランチには、正常な(指摘事項が無い)状態にしておきます。

### GitHub Access tokenの取得

Machine Accountとして登録したアカウントのtokenを取得すべきですが、[個人アカウントのtoken](https://github.com/settings/tokens)を取得します。

### TravisCIの設定

該当のコードベースをCIするための設定を行います。
travis gemを使ってCLIで操作していきます。

```
# ログイン
% bundle exec travis login
We need your GitHub login to identify you.
This information will not be sent to Travis CI, only to api.github.com.
The password will not be displayed.

Try running with --github-token or --auto if you don't want to enter your password anyway.

Username: mediba-kitada

# 有効化
% bundle exec travis enable --org --repo mediba-Kitada/evilbot-js
mediba-Kitada/evilbot-js: enabled :)

# GitHub Access tokenを暗号化して環境変数に登録
% bundle exec travis env set GITHUB_ACCESS_TOKEN hoge --org --repo mediba-Kitada/evilbot-js
[+] setting environment variable $GITHUB_ACCESS_TOKEN
# 環境変数一覧を確認
% bundle exec travis env list --org --repo mediba-Kitada/evilbot-js
# environment variables for mediba-Kitada/evilbot-js
GITHUB_ACCESS_TOKEN=[secure]
```

### TravisCIの設定ファイルを用意する

リポジトリの直下にCIするためのyaml形式のファイルを用意します。
ビルド対象の言語は、Node.jsとしておきます。

```
# yamlファイル生成
% bundle exec travis init node_js --org --repo mediba-Kitada/evilbot-js
.travis.yml file created!
mediba-Kitada/evilbot-js: enabled :)

# yamlファイルにビルドに必要な処理を記述
% vi .travis.yml

# 言語にNode.jsを指定
language: node_js
# TravisCIのコンテナには、shllow cloneされるので、diffを取るためにリモートブランチを指定し、fetchしておく
before_script:
  - TRAVIS_FROM_BRANCH="travis_from_branch"
  - git branch $TRAVIS_FROM_BRANCH
  - git checkout $TRAVIS_FROM_BRANCH
  - git fetch origin master
  - git checkout -qf FETCH_HEAD
  - git checkout master
  - git checkout $TRAVIS_FROM_BRANCH
# ビルド時のコマンドを指定(今回はリントのみ)
script: yarn lint index.js
# ビルドに失敗した場合の処理
after_failure:
  # CI環境でSaddlerを稼働させる
  - gem install bundle
  - bundle install --without local
  # git diffの結果をリントし、checkstyleフォーマットを出力、Saddlerがパース、Pull Requestにコメント
  - git diff --name-only --diff-filter=ACMR master | grep js | xargs yarn lint | bundle exec checkstyle_filter-git diff master | bundle exec saddler report --require saddler/reporter/github --reporter Saddler::Reporter::Github::PullRequestReviewComment

# .travis.ymlのリント
% bundle exec travis lint
```

__diff対象のブランチは、origin/masterとなっていますが、実際のプロジェクトではTravisCIが用意してくれている環境変数を駆使しながら、差分ファイルを特定する処理が必要になると思います。__


### コード規約に違反するPull Requestを投げる

ビルドログを```tail -f```出来たりします。

```
% bundle exec travis logs
```

Pull Requestを確認してみましょう。

指摘事項をコメントしてくれてますね。

## おわりに

今回は、開発チームが私含め全員JavaScript初心者という男前なプロジェクトでしたので、自動化や省力化にはそれなりのリソースを割きました。
Saddlerの導入は、開発の後期に実施したのですが、もう少し早いタイミングで実施出来ればと思いました。
我々人類が質の高いコードレビューを行うために、機械に働いてもらうべきですね。

## 参考

[Travis CIでtextlintの指摘をPull Requestのレビューコメントとして書き込む - Qiita](http://qiita.com/azu/items/c2305f3dded3fda968e0)
[git リポジトリの最新の履歴だけを取得する shallow clone - Qiita](http://qiita.com/usamik26/items/7bfa61b31344206077fb)
[悪いロボット | 辺境社会研究室](http://youkoseki.tumblr.com/post/126911561050/bad-robots)
