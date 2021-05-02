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
すると以下画像の画面が表示されますので赤枠の中の値をそれぞれ入力してください。

![RLogin1](img/RLogin1.png)

「SSH認証鍵」のボタンを押下すると以下の画面が開かれるため  
`vagrant_test_manual/.vagrant/machines/default/virtualbox` 下の `private_key` を指定して開くボタンを押しましょう。

![RLogin2](img/RLogin2.png)

以上で設定が完了したので、設定した接続情報を選択して「OK」ボタンでゲストOSにログインしましょう。

※初回ログイン時に注意が表示されますが、OKを押してください。

ログイン後は以下のような文言が表示されるかと思います。

```
[vagrant@localhost ~]$
```

これからホストOS・ゲストOSを操作していくにあたり、インストール等を行っていくこととなりますので、自身が操作しているOSがどちらかについて間違いが無いよう注意しましょう。

## パッケージのインストール

グループパッケージをインストールし、gitなどの開発に必要なパッケージを一括でインストールしましょう。  
下記コマンドを実行してください。

```shell
$ sudo yum -y groupinstall "development tools"
```
***

## PHPのインストール

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

## Composerのインストール

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

## データベースのインストール

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