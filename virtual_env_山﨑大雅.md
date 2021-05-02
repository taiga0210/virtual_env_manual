# 環境構築手順書
***
__使用するソフトウェア__  
* PHP 7.3  
* Nginx  
* MySQL 5.7  
* Laravel 6.0

__OS__  
* CentOS  
***

## 導入・条件

本手順書の開始は「Vagrantディレクトリの作成」となっているので、

* VirtualBox ver6.0.14のインストール  
* Vagrantの最新バージョンのインストール（2021/5/2時点では2.2.15）  
* vagrant boxのダウンロード（Linux/CentOS/バージョン7）

は完了している状態からの手順となります。

また、ゲストOSの操作環境は[RLogin](http://nanno.dip.jp/softlib/man/rlogin/#INSTALL)を使用しています。

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
上記のコマンドで起動ができ、```vagrant halt```で停止することが可能です。

