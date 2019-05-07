# centos7_lamp_configs
Samples of configuration files for LAMP environment on CentOS 7

For more information, please see https://docs.google.com/document/d/12VbZYST8h0rxLE18c0xdS_PHvkq-AcLYVa-ONuPe5dU

# CentOS 7.x LAMP環境構築マニュアル
インストール用のOSイメージをダウンロード

## CentOS 7

- http://ftp.nara.wide.ad.jp/pub/Linux/centos/7/isos/x86_64/
- ディスクイメージ：http://ftp.nara.wide.ad.jp/pub/Linux/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-1503-01.iso
時間がもったいないので、minimalをダウンロード。

CentOS 7はそれまでのCentOSからかなり変更されている。

## VirtualBoxの準備

Machine -> Newで仮想マシンの作成を開始し、以下を設定してCreateをクリック。

| 項目名 | 設定例 |
| ------ | ------ |
| Type | Linux |
| Version | CentOSの64bitの場合はRed Hat (64-bit)を選択 |
| Memory | 1GB以上 |
| Hard disk | Create a virtual hard disk nowを選択 |

仮想ディスクの作成ダイアログが出てくるので以下の設定でCreateをクリック。

| 項目名 | 設定例 |
| ------ | ------ |
| 保存先 | ゲストマシンの容量を確認して適切なディスク上に指定する。 |
| File size | 20GB以上 |
| Hard disk | VDI |
| Memory | 1GB以上 |
| Storage on physical hard disk|どちらでも良い。<br>Fixed sizeのほうがパフォーマンスはいいらしい。 |

出来上がった仮想マシンを右クリックして「Settings」を開き、OKをクリック。

| 項目名 | 設定例 |
| ------ | ------ |
| Storage | Controller: IDE<br>Emptyをクリックし、右のAttributesのOptical DriveのCDマークをクリックし、Choose Virtual Oprical Disk Fileを選択し、ダウンロードしたCentOSのディスクイメージを選択する。 |
| Network | Adapter 1<br>Attached to: Bridged Adapterを選択する。<br>Host-only adapterでもよい。その場合は192.168のIPになる。 |

仮想マシンをダブルクリックして起動。


## CentOSのインストール

Install CentOS 7を選択してインストーラーを起動。
インストールディスクのチェックはSKIPする。
インストラーの手順にしたがってインストール（最小限のインストール）し、再起動する。

| 項目名 | 設定例 |
| ------ | ------ |
| ソフトウェア | 最小限のインストール |
| ネットワークとホスト名 | 「オン」にして、ホスト名に任意の名称を入力。 |
| インストール先 | 内容を確認して「完了」をクリック。 |

以上が終わると、「インストール開始」ボタンが有効になる。
インストール実行中にrootユーザーのパスワードを設定する。「完了」ボタンは2回クリックしないと行けないので注意。
最小構成のインストールの場合は数分で終わる。

以降、特に指定のない限りはrootユーザーで行うこと。




## CentOSの初期設定

### ネットワークの設定

ネットワークはインストール時に有効になっているので、特に設定することはない。
ipコマンドでIPアドレスを確認しておく。

```bash
[root@centos7 ~]# ip addr
```

### ログイン用ユーザーを作成

```bash
[root@centos7 ~]# useradd webmaster
[root@centos7 ~]# passwd webmaster
```

SSHはデフォルトで設定されているので、移行はSSHクライアントで接続して作業する。

- SSHクライアント
  - PuTTY
  - Tera Term

### 最低限必要なパッケージのインストールと設定

```bash
[root@centos7 ~]# yum install ntp vim wget zip unzip
[root@centos7 ~]# vi /etc/bashrc
以下を最後の行の1つ前に追加
alias ll='ls -la'
alias la='ls -A'
export LANG=C
export EDITOR=vim
export LS_COLORS=$LS_COLORS':di=33'
alias vi='vim'
----------
[root@centos7 ~]# vi /etc/vimrc

以下を最後の行に追加
" ORIGINAL SETTINGS
set nowrap
set nu
set ic
hi Comment ctermfg=2

set noswapfile
set nobackup

" Indents
function SetIndentWeb()
    set tabstop=2
    set softtabstop=2
    set shiftwidth=2
endfunction

set tabstop=4
set softtabstop=4
set shiftwidth=4
autocmd BufNewFile,BufRead *.html,*.css,*.js :call SetIndentWeb()

set autoindent
set smartindent
set expandtab
"set noexpandtab

set iminsert=0
set imsearch=0
----------
[root@centos7 ~]# source /etc/bashrc
```

### リポジトリの追加

以下のリポジトリはよく使うのでインストールしておく。

- EPEL (https://fedoraproject.org/wiki/EPEL/ja)
  - インストール方法：https://fedoraproject.org/wiki/EPEL/FAQ#howtouse
  - CentOS 7の場合、EL7の手順に従う。
  - Fedoraプロジェクトのリポジトリ。
- remi repo（http://rpms.famillecollet.com/）
  - インストール方法：http://blog.remirepo.net/pages/Config-en
  - CentOS 7の場合、Enterprise Linux 7の手順に従う。
  - PHPの更新が頻繁に行われているリポジトリ。
- ~~rpmforge (http://pkgs.repoforge.org/rpmforge-release/)~~
  - ~~インストール方法：https://wiki.centos.org/AdditionalResources/Repositories/RPMForge#head-f0c3ecee3dbb407e4eed79a56ec0ae92d1398e01~~
- MySQL Yum Repository (http://dev.mysql.com/downloads/repo/yum/)
  - インストール方法：http://dev.mysql.com/doc/mysql-yum-repo-quick-guide/en/
  - MySQLのオフィシャルyumリポジトリ。
  - 2015年10月現在、MySQL 5.5、5.6、5.7（デフォルト）がインストール可能。

remi repoはデフォルトでは無効なので、有効なリポジトリにしておく。

```bash
[root@centos7 ~]# vi /etc/yum.repos.d/remi.repo
  5 [remi]
  9 enabled=1
 12
 22 [remi-php56]
 27 enabled=1
```

まずは基本システムのアップグレード

```bash
[root@centos7 ~]# yum update
```


## LAMP環境の構築

### LAMP環境作成に必要なパッケージのインストール

```bash
[root@centos7 ~]# yum install httpd php php-bcmath php-captchaphp php-cli php-common php-dba php-devel \
php-apc php-embedded php-enchant php-fpdf php-fpdf-doc php-fpm php-gd php-geshi php-imap php-interbase \
php-intl php-ldap php-libdmtx php-magpierss php-mbstring php-mcrypt php-mysqlnd php-oauth php-odbc \
php-opcache php-pdo php-pear php-pecl-imagick php-pecl-memcache php-pecl-memcached php-pecl-zip php-pgsql \
php-php-gettext php-process php-pspell php-recode php-shout php-simplepie php-snmp php-soap php-tidy \
php-xml php-xmlrpc php-yaml php-zipstream mysql-server mysql-client memcached samba mailx
```

#### Apacheをインストールする場合

```bash
[root@centos7 ~]# yum install httpd
```

#### nginxをインストールする場合

```bash
[root@centos7 ~]# yum install nginx php-fpm
```

### 基本フォルダの設置

```bash
[root@centos7 ~]# chown -R webmaster:webmaster /var/www
```

### セキュリティの設定

まずSELINUXの設定を切る。

```bash
[root@centos7 ~]# cp /etc/selinux/config /etc/selinux/config.orig
[root@centos7 ~]# setenforce 0
[root@centos7 ~]# getenforce
Permissive
[root@centos7 ~]# vi /etc/selinux/config
SELINUX=disabled
```

次にfirewalldをOFFにする。
CentOS 7ではiptablesに変わり、アクセス制限にfirewalldを使用している。

```bash
[root@centos7 ~]# systemctl stop firewalld
[root@centos7 ~]# systemctl disable firewalld
```

ここで一旦rebootして設定を反映する。


## NTP（時刻）の設定

設定を書き換える前にオリジナルをバックアップ。

```bash
[root@centos7 ~]# cp /etc/ntp.conf /etc/ntp.conf.orig
```

- INTERNET MULTIFEED CO. (http://www.jst.mfeed.ad.jp/)
  - セットアップ方法：http://www.jst.mfeed.ad.jp/about/04.html

NTPの設定は以下を参考にする。

- https://github.com/tomgoodsun/centos7_lamp_configs/blob/master/config/etc/ntp.conf

```bash
[root@centos7 ~]# wget https://raw.githubusercontent.com/tomgoodsun/centos7_lamp_configs/master/config/etc/ntp.conf -O /etc/ntp.conf 
```

ntp.confを書き換えたらntpdを起動または再起動。

```bash
[root@centos7 ~]# systemctl enable ntpd
[root@centos7 ~]# systemctl start ntpd
```

## Samba（Windowsファイル共有）の設定

設定を書き換える前にオリジナルをバックアップ。

```bash
[root@centos7 ~]# cp /etc/samba/smb.conf /etc/samba/smb.conf.orig
```

Sambaの設定は以下を参考にする。

- https://github.com/tomgoodsun/centos7_lamp_configs/blob/master/config/etc/samba/smb.conf

```bash
[root@centos7 ~]# wget https://raw.githubusercontent.com/tomgoodsun/centos7_lamp_configs/master/config/etc/samba/smb.conf -O /etc/samba/smb.conf 
```

設定を書き換えたらsmbを起動または再起動。

```bash
[root@centos7 ~]# systemctl enable smb
[root@centos7 ~]# systemctl start smb
[root@centos7 ~]# systemctl enable nmb
[root@centos7 ~]# systemctl start nmb
```

Windowsからアドレスバーに以下を入力して共有ディレクトリが閲覧出来るか確認。
またファイルの作成、内容の変更、削除が正常に行われるか確認する。

`\\{サーバーのIPアドレス}`



## PHPのセットアップ

設定を書き換える前にオリジナルをバックアップ。

```bash
[root@centos7 ~]# cp /etc/php.ini /etc/php.ini.orig
```

PHPの設定は以下を参考にする。
- https://github.com/tomgoodsun/centos7_lamp_configs/blob/master/config/etc/php.ini

## ウェブサーバーのセットアップ

### Apacheの場合

設定を書き換える前にオリジナルをバックアップ。

```bash
[root@centos7 ~]# cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.orig
```

ディレクトリを作る。

```bash
[root@centos7 ~]# mkdir -p /etc/httpd/vhosts /var/www/vhosts /var/www/logs/httpd /var/www/session
[root@centos7 ~]# chown -R webmaster:webmaster /var/www
```

mod_phpの設定を変更する。
デフォルトの`session.save_path`は`/var/lib/php/session`となっているので、Apacheの耳垢ユーザーでは書き込めない。`/var/www/session`に書き換える。

```bash
[root@centos7 ~]# vi /etc/httpd/conf.d/php.conf
 60     #php_value session.save_path    "/var/lib/php/session"
 61     php_value session.save_path    "/var/www/session"
```

Apache 2.4ではNot Foundの場合に`/usr/share/httpd/noindex/`のファイルが使われるので、使われないようにする。

```bash
[root@centos7 ~]# mv /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/welcome.conf.bk
```

Apacheの設定は以下を参考にする。

- https://github.com/tomgoodsun/centos7_lamp_configs/blob/master/config/etc/httpd/conf/httpd.conf

またApache 2.4からは設定ファイルが細かく分割されている。デフォルトで問題となりそうなのはiconsディレクトリ。常にエラーログに記録されることが気になるなら以下のファイルのalias設定をコメントアウトすることをおすすめする。

```bash
[root@centos7 ~]# vi /etc/httpd/conf.d/autoindex.conf
 21 #Alias /icons/ "/usr/share/httpd/icons/"
```

設定を書き換えたらApacheの設定ファイルの書式を確認して、起動または再起動。

```bash
[root@centos7 ~]# systemctl enable httpd
[root@centos7 ~]# apachectl configtest
[root@centos7 ~]# systemctl start httpd
```

### nginxの場合

設定を書き換える前にオリジナルをバックアップ。

```bash
[root@centos7 ~]# cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.orig
[root@centos7 ~]# cp /etc/php-fpm.conf /etc/php-fpm.conf.orig
[root@centos7 ~]# cp /etc/php-fpm.d/www.conf /etc/php-fpm.d/www.conf.orig
```

nginxの設定は以下を参考にする。

- https://github.com/tomgoodsun/centos7_lamp_configs/blob/master/config/etc/nginx/nginx.conf

php-fpmの設定は以下を参考にする。

- https://github.com/tomgoodsun/centos7_lamp_configs/blob/master/config/etc/php-fpm.conf
- https://github.com/tomgoodsun/centos7_lamp_configs/blob/master/config/etc/php-fpm.d/www.conf

```bash
[root@centos7 ~]# systemctl enable nginx
[root@centos7 ~]# systemctl start nginx
[root@centos7 ~]# systemctl enable php-fpm
[root@centos7 ~]# systemctl start php-fpm
```


## MySQL Serverのセットアップ

ここではMySQL 5.7のセットアップ方法を紹介します。

設定を書き換える前にオリジナルをバックアップ。

```bash
[root@centos7 ~]# cp /etc/my.cnf /etc/my.cnf.orig
```

MySQL 5.7以降ではrootのデフォルトパスワードが勝手に生成されてしまう。
`/root/.mysql_secret`に記載されているとのこと。ない場合は`/var/log/mysql.log`に記載が残っているので確認してみる。
また5.6からパスワードポリシーが厳しくなっている。開発環境用はあまり厳しすぎると使いにくいので、かなりゆるく設定する。

MySQLの設定は以下を参考にする。（パスワードに対する制限をかなりゆるくしている）
MySQL 8.0からパスワード関連の変数名が変わっていいるので注意。

- https://github.com/tomgoodsun/centos7_lamp_configs/blob/master/config/etc/my.cnf

またはrootでの操作はデフォルトのパスワードを変更した後でないと出来ないようになっているので、まずrootでログインした後はパスワードを変更することから始めなければならない。

```bash
[root@centos7 ~]# systemctl enable mysqld
[root@centos7 ~]# systemctl start mysqld
[root@centos7 ~]# mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.9 MySQL Community Server (GPL)

Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> SET PASSWORD FOR root@localhost=PASSWORD('password');
Query OK, 0 rows affected, 1 warning (0.00 sec)

MySQL 8.0ではSET PASSWORD FORは使えない。代わりにALTER USERを使う。
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'password';


データベースとユーザーの作り方に関しては従来の方法で行える。
[root@centos7 ~]# mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.9 MySQL Community Server (GPL)

Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> SHOW VARIABLES LIKE 'validate_password%'; # 環境設定の確認（パスワード系の設定をゆるくしておく）
+--------------------------------------+-------+
| Variable_name                        | Value |
+--------------------------------------+-------+
| validate_password_check_user_name    | OFF   |
| validate_password_dictionary_file    |       |
| validate_password_length             | 4     |
| validate_password_mixed_case_count   | 0     |
| validate_password_number_count       | 0     |
| validate_password_policy             | LOW   |
| validate_password_special_char_count | 0     |
+--------------------------------------+-------+
7 rows in set (0.00 sec)

mysql> CREATE DATABASE mydb CHARACTER SET utf8;
Query OK, 1 row affected (0.00 sec)

mysql> CREATE USER mydb_user@'%' IDENTIFIED WITH mysql_native_password BY 'password';
Query OK, 0 rows affected (0.01 sec)

mysql> GRANT ALL PRIVILEGES ON mydb.* TO mydb_user@'%';
Query OK, 0 rows affected (0.00 sec)

mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW WARNINGS;
Empty set (0.00 sec)

mysql>
```
ポイントはMySQL 5.6までは`GRANT ALL PRIVILEGES ON mydb.* TO mydb_user@'%' IDENTIFIED WITH mysql_native_password BY 'password';`という構文が使えていたが
MySQL 5.7では使用できなくなっていること。

その代わり`CREATE USER`文が使えるで、それを使って設定する。あまりないかもしれないが`CREATE USER`文を使用してパスワードのハッシュアルゴリズムを指定出来るようになっているのだが、変に設定してしまうとPHPとかから接続できなくなってしまうので注意が必要。

またエラーが発生しているかどうかは`SHOW WARNINGS;`で見ることが出来る。


## 自動起動設定

以下のデーモンはsystemctlコマンドで自動起動設定（enable）をしておく。
自動起動設定はOS起動時に自動的に立ち上がるソフトウェアを指定すること。

- httpd
- mysqld
- ntpd
- samba
- memcached


## 最後の設定

ここで一旦rebootして、すべて正常に起動できるかを確認する。

```bash
[root@centos6 ~]# reboot
```

