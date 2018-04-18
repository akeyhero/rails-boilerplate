# boilerplate

これは、 Ruby on Rails で開発を開始するために必要な環境を自動で準備するための秘伝のタレです。

README には Rails 起動に必要な手順を記しています。それ以上の作業については、 [Wiki](https://github.com/akeyhero/rails-boilerplate/wiki) に記述していきます。

## 1. Vagrant & VirtualBox の導入

Vagrant & VirtualBox をなんとなく導入してください。例) [https://qiita.com/ohuron/items/057b74a42b182b200ae6](https://qiita.com/ohuron/items/057b74a42b182b200ae6)

Mac の場合は HomeBrew Cask でインストールできます。

```bash
$ brew cask install vagrant
$ brew cask install virtualbox
```

Windows の場合、加えて SSH をインストールする必要があります: https://www.eaton-daitron.jp/techblog/4627.html

## 2. Vagrantfile の準備

適当なフォルダを作って、 `Vagrantfile` をコピーします。

## 3. 環境構築

ターミナルで以下のコマンドを実行します。これは結構時間がかかります。

```bash
$ vagrant up
```

ターミナルは、 Mac の場合標準のものか iTerm2 、 Windows の場合、最初から入っているものでは PowerShell がオススメです。あとからインストールする場合は [Mintty](http://mintty.github.io/) がいいらしいですが、知見が全くありません。

## 4. 起動・ログイン

ログインは開発のたびにやることになります。

```bash
$ # 起動してない場合
$ vagrant up
$ # SSH ログイン
$ vagrant ssh
```

`cd` コマンド (change directory) で、 `/vagrant` に移動します。

```bash
$ cd /vagrant
```

このディレクトリは、 `vagrant up` を実行したフォルダと同期しています。

## 5. プロジェクトディレクトリの作成

### GitHub や BitBucket で開発する場合

GitHub などでリポジトリを作成します。リポジトリ名は、特に設定しなければ自動的に Rails のプロジェクト名にもなります。作成したら、GitHub の場合、右上に緑色の「Clone or download」から URL を取得し、 `git clone` を実行します。

```bash
$ git clone $(リポジトリの URL)
```

リポジトリ名と同名のディレクトリが作成されるので、そのディレクトリに移動します。

```bash
$ cd $(リポジトリ名)
```

### ローカルリポジトリで開発する場合

プロジェクトのディレクトリを作成し、そのディレクトリに移動します。

```bash
$ mkdir $(プロジェクト名)
$ cd $(プロジェクト名)
```

ローカルリポジトリとして初期化します。

```bash
$ git init
```

## 6. Ruby on Rails のインストール

Rails 開発では bundler というパッケージ管理システムを使います。アプリケーション開発の世界では、他人が既に作ったものをわざわざ自作することを「車輪の再発明」と言って揶揄します。「車輪」はライブラリとして RubyGems (gem) で管理されています。アプリケーション開発においては何十、何百といった gem を使います。これらのバージョンが食い違うといとも簡単に動かなくなるので、それを管理する仕組みであるところの bundler が必要になるわけです。(ちなみに、 gem もパッケージ管理システムの一種だが、より低レイヤー)

ログインしたターミナル上で、

```bash
$ bundle init
```

を実行すると、 `Gemfile` というファイルが生成されます。ここに利用する gem を列挙していくことになります。`Gemfile` を開き、最後の行の `# gem "rails"` の `# ` を消します。

```ruby
# frozen_string_literal: true

source "https://rubygems.org"

git_source(:github) {|repo_name| "https://github.com/#{repo_name}" }

gem "rails"
```

そして、先程のターミナルで以下のコマンドを実行すると、 Ruby on Rails がインストールされます。

```bash
$ bundle install --path=vendor/bundle --without=production --jobs=4
```

参考文献: 細かい話は[こちらの記事](http://maetoo11.hatenablog.com/entry/2016/03/04/144216)に掲載されています。

## 7. Ruby on Rails プロジェクトの作成

### rails new

以下のコマンドを実行します。 `bundle exec` は bundler でインストールした gem を動かすための枕詞です。途中で `conflict Gemfile` と表示される事になりますが、 `Y` と入力して上書きしてください。

```bash
$ bundle exec rails new . --database=postgresql --webpack=vue --skip-coffee --skip-turbolinks --skip-git --skip-test-unit
```

Vagrant で実行する場合、 Vagrant のローカル IP アドレスをホワイトリストに登録する必要があります。 `config/environments/development.rb` に以下の設定を書き加えます。

```ruby
  config.web_console.whitelisted_ips = '10.0.2.0/24'
```

### RSpec 導入

ここではテストをスキップしましたが、これは標準の minitest ではなく、 RSpec を導入するためです。(ある程度の規模 Rails アプリケーションをテスト無しで開発することはできません。) Gemfile を開き、以下のコードを追加します。

```ruby
group :test, :development do
  gem 'rspec-rails', '~> 3.7.2'
end
```

インストール & セットアップをします。

```bash
$ bundle install
$ bundle exec rails generate rspec:install
```

### DB 作成

DB を作成します。

```bash
$ bundle exec rails db:create
```

### 動作確認

以下のコマンドを実行し、 `* Listening on tcp://0.0.0.0:3000` と表示されたら、 [http://localhost:3000](http://localhost:3000) にアクセスします。

```bash
$ bundle exec rails server
```

`Yay! You’re on Rails!` が表示されたら完了です。 Enjoy!
