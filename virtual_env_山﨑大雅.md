# 環境構築手順書

***
__使用するソフトウェア__  
* PHP 7.3  
*  Nginx  
* MySQL 5.7
* Laravel 6.0  

__OS__
* CentOS
***

## 導入・条件

本手順書は「Windows10」をホストOSとする場合の手順書となります。

### VirtualBoxのインストール

__※今回使用するVagrantの最新バージョンがVirtualBoxの最新バージョンに対応していないため、VirtualBoxはver6.0.14をインストールします。__

下記サイトにて「ver6.0.14」の「Windows hosts」を選択しダウンロードしてください。

[Virtual Box公式](https://www.virtualbox.org/wiki/Download_Old_Builds_6_0)

ダウンロード後にデスクトップ画面に「Oracle VM VirtualBox」のアイコンがあればインストール完了しています。
***

### Vagrantインストール

下記サイトからインストールします。

[Vagrant公式](https://www.vagrantup.com/)

インストール後に下記コマンドを実行し問題なくインストールができているかの確認を行います。

```
vagrant -v
```
バージョンの確認できたらインストール完了です。
***

### RLoginのインストール

WindowsのコマンドプロンプトにはSSHコマンドが使用できないため、ターミナルソフトを導入しゲストOSの操作を行うようにします。  
下記サイトよりインストールしましょう。

[RLogin](http://nanno.dip.jp/softlib/man/rlogin/#INSTALL)
***

### Vagrantダウンロード

仮想環境を構築する際の元になるOSを決めていきます。  
今回はVagrantを導入していますので下記の流れを進め __LinuxのCentOSのバージョン7__ を使用できるよう指定しましょう。

```
vagrant box add centos/7
# 上記実行後下記の選択肢が表示される
1) hyperv
2) libvirt
3) virtualbox
4) vmware_desktop

Enter your choice: 3
```

今回使用するソフトは __VirtualBox__ のため、 __3__ を入力しEnterを押しましょう。  
下記のように表示されたら完了です。

```
Successfully added box 'centos/7' (v1902.01) for 'virtualbox'!
```
***

##  Vagrantの作業ディレクトリを用意する

__Vagrant_test_manual__ という名前のディレクトリを作成します。  
※このディレクトリの中に設定等の重要なファイルを格納することとなるので、作成後は移動や削除を行わないようにしてください。

「自分の作業用ディレクトリ」もしくは「デスクトップ」の階層で、以下コマンドを実行してください。
```
mkdir vagrant_test_manual && cd vagrant_test_manual
```
__vagrant_test_manual__ というディレクトリが作成され、そのディレクトリへの移動を行えたので、その状態で下記コマンドを実行します。

```
vagrant init centos/7
```
実行後問題なければ以下のような文言が表示されます。
```
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
```

## Vagrantfileの編集とVagrantの起動

### Vagrantfileの編集

作成した __vagrant_test_manual__ ファイル直下にある「Vagrantfile」を編集します。
今回編集を行うのは3点です。

※テキストエディタで開き編集してください。

```Vagrantfile
# 変更点①
config.vm.network "forwarded_port", guest: 80, host: 8080
```
上記の1点は文頭の「＃」を外しコメントインしてください。
また、以下の2点も同じくコメントインし、内容を編集してください。
```
# 変更点②
config.vm.network "private_network", ip: "192.168.33.10"
# ↓ 以下に編集
config.vm.network "private_network", ip: "192.168.33.19"

# 変更点③
config.vm.synced_folder "../data", "/vagrant_data"
# ↓ 以下に編集
config.vm.synced_folder "./", "/vagrant", type:"virtualbox"
```
上記の3点を行うことで使用するIPや、ホストOSとゲストOSを同期をリアルタイムで行う設定が完了しました。

***

### Vagrantの起動

仮想環境を構築する準備が整いましたので起動をしていきましょう。  
Vagrantfileがあるディレクトリ（Vagrant_test_manual）にて以下のコマンドを実行しましょう。
```
vagrant up
```
上記のコマンドで起動ができ、`vagrant halt`で停止することが可能です。

***

## ゲストOSへのログイン

前回の`vagrant up`のコマンドによりゲストOSが起動しました。
今回はそのゲストOSへ、RLoginを使用しログインしましょう。

### RLoginの設定

RLoginを起動後、ログイン先を選択する画面が開くので、今回起動させたゲストOSを追加するために「新規」ボタンを押下しましょう。

以下の項目を入力しましょう。

・ホスト名　⇒　`127.0.0.1`と入力  
・TCPポート　⇒　`2222`と入力  
・ログインユーザ名　⇒　`vagrant`と入力  

入力後、同画面内にある「SSH認証鍵」のボタンを押下するとファイルの選択画面が開かれるため  
`vagrant_test_manual/.vagrant/machines/default/virtualbox` のパスの直下にある `private_key` を指定して開くボタンを押しましょう。

以上で設定が完了したので、設定した接続情報を選択して「OK」ボタンでゲストOSにログインしましょう。

※初回ログイン時に注意が表示されますが、OKを押してください。

ログイン後は以下のような文言が表示されるかと思います。

```
[vagrant@localhost ~]$
```

これからホストOS・ゲストOSを操作していくにあたり、インストール等を行っていくこととなりますので、自身が操作しているOSがどちらかについて間違いが無いよう注意しましょう。

ホストOSは通常のコマンドプロンプトでの作業となり、ゲストOSはRLoginでの作業になります。

***

## 各種ソフトのインストール

ここでは開発に必要なソフトから、Laravelのもとになる**PHP**、PHPの管理パッケージである**Composer**、使用するデータベースの**MySQL**をインストールします。

### パッケージのインストール

グループパッケージをインストールし、gitなどの開発に必要なパッケージを一括でインストールしましょう。  
下記コマンドを実行してください。

```shell
$ sudo yum -y groupinstall "development tools"
```
***

### PHPのインストール

今回の最終目標のLaravelの動作をさせるためには、PHPのバージョンを7以上にする必要があるため、外部パッケージツールをダウンロードしてからPHPをインストールしていきます。  
PHP7.3をダウンロードするため、下記コマンドを順番に実行していきましょう。

```shell
$ sudo yum -y install epel-release wget
$ sudo wget http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
$ sudo rpm -Uvh remi-release-7.rpm
$ sudo yum -y install --enablerepo=remi-php73 php php-pdo php-mysqlnd php-mbstring $ php-xml php-fpm php-common php-devel php-mysql unzip
$ php -v
```

上記の5つのコマンドを実行しPHPのバージョンが確認できたらPHPのインストールは完了です。
***

### Composerのインストール

PHP管理パッケージであるcomposerをインストールします。  
下記コマンドを順番に実行してください。

```shell
$ php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
$ php composer-setup.php
$ php -r "unlink('composer-setup.php');"

# どのディレクトリにいてもcomposerコマンドを使用できるようfileの移動を行います
$ sudo mv composer.phar /usr/local/bin/composer
$ composer -v
```

最後のバージョン確認コマンドにより、composerのバージョンが確認できたらインストール完了です。
***

### MySQLのインストール


今回インストールするデータベースはMySQL 5.7となります。

centos7は、デフォルトでmariaDBというデータベースがインストールされていますが、MariaDBはMySQLと互換性があるDBなので気にせず、MySQLのインストールを進めていきます。

下記コマンドを順番に実行しましょう。

```shell
$ sudo wget https://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm
$ sudo rpm -Uvh mysql57-community-release-el7-7.noarch.rpm
$ sudo yum install -y mysql-community-server
$ mysql --version
```

バージョンの確認ができたらインストール完了です。
次にMySQLを起動し接続を行います。下記コマンドを実行しましょう。

```shell
$ sudo systemctl start mysqld
```

これでMySQLに接続が可能となりました。
***

## MySQLの設定とDBの作成

インストールしたデータベースのパスワード設定を行うため、実際にMySQLに接続し、今回使用するデータベースを作成していきましょう。

### MySQLのパスワードの再設定

今回はMySQLのrootユーザーにデフォルトでパスワードがかかっていますので、パスワードを調べ接続後にパスワードの再設定を行う必要があります。

※ 今回は、比較的簡単な方法でパスワードの再設定を行いますが、セキュリティ的によろしくはないため本番環境と呼ばれる環境でこの方法で再設定するのは避けてください。

下記コマンドを実行しましょう。

```shell
$ sudo cat /var/log/mysqld.log | grep 'temporary password'
```

実行後下記のような表示がされ、`hogehoge`と表示されている場所に記載されている文字列が現在のパスワードとなります。

```shell
2017-01-01T00:00:00.000000Z 1 [Note] A temporary password is generated for root@localhost: hogehoge
```

#### _grep_
`grep`は左辺の実行結果の中から、grepで指定した文字列と一致する行や文字列を持つファイルやディレクトリのみを出力するコマンドです。他のコマンドと組合わせて右辺に使用します。  
例えば `ls | grep XXX` とした場合は 現在のディレクトリに XXX とマッチするディレクトリやファイル名があればそれらのみを表示します。

#### |
`|`  はパイプラインといって  `左辺 | 右辺`  と指定してコマンドを実行することで、左辺の実行結果を右辺に引き渡して右辺を実行することができます。

先程の  `cat /var/log/mysqld.log | grep 'temporary password'`  は左辺で  
`/var/log/mysqld.log`  というファイルを出力した結果を右辺に渡し、右辺ではその結果から  `temporaray password`  という文字列を持つ行のみを出力しました。
***
先ほど出力した文字列のパスワードをコピー後、以下のコマンドを実行し、パスワード入力時にペーストしてください。

```
$ mysql -u root -p
Enter password:
mysql >
```

問題なく接続ができたらパスワードの再設定を行います。下記のコマンドを実行しましょう。

```mysql
mysql > set password = "新たなpassword";
```

これでMySQLのログイン及びパスワードの再設定が完了しました。
***

## データベースの作成

実際にLaravelのTodoアプリケーションを動かす上で使用するデータベースの作成を行います。

```mysql
mysql > create database laravel_app_manual;
```

Query OKと表示されたら作成は完了となります。
***

## Nginxのインストールとブラウザ確認

今回使用するWebサーバーをインストールし、実際にブラウザに表示してみます。

### Nginxのインストール

今回の使用するWebサーバをインストールしていきます。
viエディタを使用し以下のファイルを作成します。

```shell
$ sudo vi /etc/yum.repos.d/nginx.repo
```

書き込む内容は以下になります。

```
[nginx]
name=nginx repo
baseurl=https://nginx.org/packages/mainline/centos/\$releasever/\$basearch/
gpgcheck=0
enabled=1
```

書き終えたら保存して、以下のコマンドを実行しNginxのインストールを実行します。

```shell
$ sudo yum install -y nginx
$ nginx -v
```

上記のコマンドでNginxのバージョンが確認できたら、起動をしましょう。
下記コマンドを実行してください。

```shell
$ sudo systemctl start nginx
```

これでNginxのインストールと起動が完了しましたが、ホストOSのブラウザにて  _[http://192.168.33.19](http://192.168.33.19/)_  と入力してもNginxのWelcomeページは表示されないかと思います。
表示をさせるために「**ファイヤーウォール**」の設定をしていきましょう。
***

### ファイヤーウォールの設定

ブラウザに表示させるため、ファイヤーウォールの設定にて80ポートを経由したhttp通信によるアクセスを許可するためのコマンドを実行します。

```shell
# ファイヤーウォールの起動
$ sudo systemctl start firewalld.service
$ sudo firewall-cmd --add-service=http --zone=public --permanent

# 新たに追加を行ったのでそれをファイヤーウォールに反映させるコマンドも合わせて実行します
$ sudo firewall-cmd --reload
```
では、一旦この状態で画面を確認してみます。

もしまだ表示できないようであれば、一度以下のコマンドを実行してください。

```shell
$ sudo systemctl restart nginx
```

Nginxのwelcome画面が表示されたでしょうか？
***

### それでも表示されなかった場合

Nginxのwelcomeページが表示されず、  **Forbidden 403**  というエラーが出た場合の対処法を以下に記載します。  

viエディタを使用してSELinuxの設定を変更します。  
「SELinux コンテキスト」の不一致によりエラーが出ているので、SELinuxを無効化します。(今回はローカル環境なので無効にしてしまっても問題ありませんが、本番環境構築のときは別のアプローチが必要です。)

```shell
$ sudo vi /etc/selinux/config
```

viエディタが開き設定ファイルが表示されるので下記の部分を探してください。

```shell
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
# enforcing - SELinux security policy is enforced.
# permissive - SELinux prints warnings instead of enforcing.
# disabled - No SELinux policy is loaded.
SELINUX=enforcing
```

上記の最終行の記述を下記のように書き換えて、保存してください。

`SELINUX=disabled`

設定を反映させるためにゲストOSを再起動する必要があるので、ゲストOSをから一度ログアウトして下記コマンドを実行してください。

```
$ exit
```
```
vagrant reload　#ホストOSでの操作
```

リロードが完了したら再度ゲストOSにログインしましょう。
RLoginを再度起動しログインしましょう。

再度Nginxを起動します。

```
$ sudo systemctl start nginx
```

これでNginxのwelcomeページが表示されているはずです。画面を確認してみてください。
***

## Laravelのセットアップ

ブラウザへの表示環境が整ったため、Laravelをインストールしていきましょう。

### 事前準備
Laravelのインストールに向けての事前準備として「**Node.js**」をインストールします。  
下記コマンドを実行しましょう。

```shell
$ sudo yum -y install epel-release
$ sudo yum -y install nodejs npm
```

インストール完了後下記コマンドでバージョンが確認できたら問題なくインストールが完了しています。

```shell
$ node -v
$ npm -v
```
***

### Laravelのインストール

事前準備が完了したらLaravelプロジェクトを作成しましょう。  
今回プロジェクトを作成するディレクトリへ下記コマンドを実行し移動します。
```shell
$ cd /vagrant
```

今回のプロジェクト名は`laravel_app_manual`というプロジェクトにするので、下記コマンドを実行し作成してください。

```shell
$ composer create-project laravel/laravel --prefer-dist laravel_app_manual 6.0
```
***

### DBへの接続を設定する

MySQLのインストール時に設定したデータベース名やパスワードを **.env** ファイルへ書き込みます。  
そうすることで作成したLaravelプロジェクトで、作成していたデータベースが使用できるようになります。

```shell
$ vi /vagrant/laravel_app_manual/.env
```

上記コマンドを実行し下記内容を編集します。

```shell
APP_NAME=Laravel
APP_ENV=local
APP_KEY=base64:Rs6WHziGChaNJGg0o1mBOidiKaFZPkKeHNt6aGamvYk=
APP_DEBUG=true
APP_LOG_LEVEL=debug
APP_URL=http://localhost

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel_app_manual      # 編集
DB_USERNAME=root
DB_PASSWORD=                 # 編集(各自設定しているパスワードを入力) 
# 省略
```

設定を編集後はMySQLの再起動を行います。


```shell
$ sudo systemctl restart mysqld
```

***

### テーブルの作成

Laravelのログイン認証の実装にあたり使用する、ユーザーが追加されるテーブルの作成を行います。

下記コマンドを順番に実行しましょう。

```shell
$ sudo systemctl start mysqld #すでにMySQLを起動していた場合は不要です
$ cd /vagrant/laravel_app_manual
$ php artisan migrate
```

以下のようなメッセージが表示されれば成功です。  
うまくいかない場合は、  
MySQLが立ち上がっているかどうか？.envファイルの記述が間違っていないか？再確認しましょう。

```
Migration table created successfully.
Migrating: 2014_10_12_000000_create_users_table
Migrated:  2014_10_12_000000_create_users_table
Migrating: 2014_10_12_100000_create_password_resets_table
Migrated:  2014_10_12_100000_create_password_resets_table
Migrating: 2019_08_19_000000_create_failed_jobs_table
Migrated:  2019_08_19_000000_create_failed_jobs_table
```

***

## Nginxの設定変更後のLaravelの表示

Laravelのインストールも完了したので、実際に画面にLaravelを表示させてみましょう。

### Nginxの設定ファイルの編集
このままでは作成したLaravelを表示させるための記述がされていないので、`.conf`ファイルを編集しましょう。

使用しているOSがCentOSの場合、`/etc/nginx/conf.d` ディレクトリ下の `default.conf` ファイルが設定ファイルとなります。

```shell
$ sudo vi /etc/nginx/conf.d/default.conf
```

下記の内容を編集します。それぞれ確認し間違えのないようにしましょう。

```nginx
server {
  listen       80;
  server_name  192.168.33.19; # 編集（今回使用しているIPアドレスを入力します）
  # ApacheのDocumentRootにあたります
  root /vagrant/laravel_app_manual/public; # 追記
  index  index.html index.htm index.php; # 追記

  #charset koi8-r;
  #access_log  /var/log/nginx/host.access.log  main;

  location / {
      #root   /usr/share/nginx/html; # コメントアウト
      #index  index.html index.htm;  # コメントアウト
      try_files $uri $uri/ /index.php$is_args$args;  # 追記
  }

  # 省略

  # 該当箇所のコメントを解除し、必要な箇所には変更を加える
  # 下記は root を除いたlocation { } までのコメントが解除されていることを確認してください。

  location ~ \.php$ {
  #    root           html;
      fastcgi_pass   127.0.0.1:9000;
      fastcgi_index  index.php;
      fastcgi_param  SCRIPT_FILENAME  /$document_root/$fastcgi_script_name;  # $fastcgi_script_name以前を /$document_root/に変更
      include        fastcgi_params;
  }

  # 省略
```

Nginxの設定ファイルの変更は、以上です。  
次に **php-fpm** の設定ファイルを編集していきます。

```shell
$ sudo vi /etc/php-fpm.d/www.conf
```

変更箇所は以下になります。

```plain
;24行目近辺
user = apache
# ↓ 以下に編集
user = nginx

group = apache
# ↓ 以下に編集
group = nginx
```

設定ファイルの変更に関しては、以上となります。  
では早速起動しましょう(Nginxは再起動になります)。

```shell
$ sudo systemctl restart nginx
$ sudo systemctl start php-fpm
```

ホストOSにて **[http://192.168.33.19](http://192.168.33.19/)** へアクセスしてみましょう。Laravelの画面が表示されたかと思います。

画面は表示されますが、以下のようなLaravelのエラーが表示されると思います。

```shell
The stream or file "/vagrant/laravel_app/storage/logs/laravel.log" could not be opened: failed to open stream: Permission denied
```

これは 先程php-fpmの設定ファイルの user と group を  `nginx`  に変更したと思いますが、ファイルとディレクトリの実行 user と group に  `nginx`  が許可されていないため起きているエラーです。

そのため、以下のコマンドを実行して  `nginx`  というユーザーでもログファイルへの書き込みができる権限を付与してあげましょう。

```shell
$ cd /vagrant/laravel_app_manual
$ sudo chmod -R 777 storage
```

***

## ログイン機能の実装

最後に、ログイン機能の実装を行います。  
今回使用しているLaravelのバージョンは**6.0**になるため、**5.8**までの実装方法の`php artisan make:auth`のコマンドとは異なります。

Laravelの6.x、7.xは**Laravel/ui**のパッケージ、8.xは**Jetstream**のパッケージで別での管理となってしまったためパッケージをインストールする必要があります。

使用しているのは6.0のため下記コマンドを実行しパッケージをインストールしましょう。  

```shell
$ cd /vagrant/laravel_app_manual
$ composer require laravel/ui:^1.0 --dev
```

これでuiコマンドが使用できるようになりましたので、下記コマンドを実行しログイン機能の実装をしてください。

```shell
$ php artisan ui vue --auth
```
コマンド実行後下記の文言が表示されたら問題なく完了しています。

```shell
Vue scaffolding installed successfully.
Please run "npm install && npm run dev" to compile your fresh scaffolding.
Authentication scaffolding generated successfully.
```

しかしこのままではJSとCSSが読み込めず、ログイン認証画面のレイアウトが崩れてしまっています。  
上記の表示にあるように、下記コマンドを**ホストOS**で 実行し必要なパッケージをインストールしましょう。

```shell
cd laravel_app_manualまでの絶対パス
npm install
npm uninstall --save-dev sass-loader
npm install --save-dev sass-loader@7.1.0
```

上記実行後、下記コマンドを**ゲストOS**で実行しましょう。

```shell
npm run dev
```

これでログイン機能の実装が完了しました。  
実際に _[http://192.168.33.19](http://192.168.33.19/)_ を開き確認しましょう。「**REGISTER**」よりログイン画面へ遷移しユーザーを作成出来たら環境構築及びLaravelの実装が完了しました。

***
