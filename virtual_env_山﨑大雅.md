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

## <font color="#006699">導入・条件</font>

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

![RLogin1](RLogin1.png)

「SSH認証鍵」のボタンを押下すると以下の画面が開かれるため  
`vagrant_test_manual/.vagrant/machines/default/virtualbox` 下の `private_key` を指定して開くボタンを押しましょう。

![RLogin2](RLogin2.png)

以上で設定が完了したので、設定した接続情報を選択して「OK」ボタンでゲストOSにログインしましょう。

※初回ログイン時に注意が表示されますが、OKを押してください。

ログイン後は以下のような文言が表示されるかと思います。

```
[vagrant@localhost ~]$
```

これからホストOS・ゲストOSを操作していくにあたり、インストール等を行っていくこととなりますので、自身が操作しているOSがどちらかについて間違いが無いよう注意しましょう。
