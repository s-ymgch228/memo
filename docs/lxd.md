# Fedora 30 上で LXD を使ってみた時のメモ
lxd/lxc を Fedora 30 上で実行して shell が触れるようにする

## install
### 0. snapd をインストールする
LXD は Ubuntu 系が主に開発しているため yum/dnf でサクッとインストールできない。
そこで、ディストリビューションに依存しないでインストールできる仕組みの snapd を使う。

snapd は dnf を使ってインストールする。
```
% sudo dnf -y install snapd
% sudo systemctl start snapd.seeded.service
```
snapd を入れた後は snapd.seeded を起動して初期化っぽいことをする。
snapd.seeded は実行完了するとそのまま exit される。

### 1. LXD をインストールする
snapd を使って lxd 本体を入れる。
```
snap install lxd
```
バージョン指定するには `--channel=3.0/stable` を lxd の後につける

lxd は lxd グループに属するアカウントにローカル接続を許可するので通常使うユーザのアカウントを lxd グループに加える
```
sudo gpasswd -a <username> lxd
```
### 2. LXDを初期化する
lxd は使う前に初期化する必要がある。
初期化は `lxd init` コマンドを使う。
```
% lxd init
Would you like to use LXD clustering? (yes/no) [default=no]: no
Do you want to configure a new storage pool? (yes/no) [default=yes]: yes
Name of the new storage pool [default=default]: default
Name of the storage backend to use (dir, lvm, zfs) [default=zfs]: dir        <===== dir にする
Would you like to connect to a MAAS server? (yes/no) [default=no]: no
Would you like to create a new network bridge? (yes/no) [default=yes]: yes
What should the new bridge be called? [default=lxdbr0]: lxdbr0
What IPv4 address should be used? (CIDR subnet notation, “auto” or “none”)
[default=auto]: auto
What IPv6 address should be used? (CIDR subnet notation, “auto” or “none”)
[default=auto]: none
Would you like LXD to be available over the network? (yes/no)
[default=no]: no　　　　　　　　　　　　　　　　　　　　<===== コンテナが直接ネットワークに接続するかどうか？
Would you like stale cached images to be updated automatically? (yes/no)
[default=yes] yes
Would you like a YAML "lxd init" preseed to be printed? (yes/no)
[default=no]: no
%
```

### 3 Linux カーネル起動時のパラメタに namespace を有効化する値を追加する
namespace はカーネルの機能を lxc ごとに分割できるように名前を付けるような機能(多分)。これは起動時のパラメタで設定する。
(そもそもこの設定が必要かどうかはわからない)
```
% sudo vi /etc/default/grub
...
GRUB_CMDLINE_LINUX="resume=/dev/mapper/fedora-swap rd.lvm.lv=fedora/root rd.lvm.lv=fedora/swap rhgb quiet user_namespace.enable=1 namespace.unpriv_enable=1"
...
```
`/etc/default/grub` の中に書かれている GRUB_CMDLINE_LINIUX に `user_namespace.enable=1` と `namespace.unpriv_enable=1` を追加する。
追加が終わったら grub の設定ファイルに反映する

```
% sudo grub2-mkconfig -o /boot/efi/EFI/fedora/grub.cfg
```
`/boot/efi/...` は UEFI boot の場合に読み込まれるフォルダ。 BIOS 起動の場合はパスが違う。
update した後は reboot して反映する。

## 使い方
### コンテナのイメージを決める
`lxc image list images:` コマンドで公開されているコンテナのイメージ一覧が見れる。
はじめはこのリストから作るコンテナのイメージを決める。


```
 $ lxc image list images: 
+--------------------------------------+--------------+--------+----------------------------------------------+---------+-----------+-------------------------------+
|                ALIAS                 | FINGERPRINT  | PUBLIC |                 DESCRIPTION                  |  ARCH   |   SIZE    |          UPLOAD DATE          |
+--------------------------------------+--------------+--------+----------------------------------------------+---------+-----------+-------------------------------+
| alpine/3.6 (3 more)                  | 41854f8f76db | yes    | Alpine 3.6 amd64 (20190619_13:00)            | x86_64  | 3.17MB    | Jun 19, 2019 at 12:00am (UTC) |
+--------------------------------------+--------------+--------+----------------------------------------------+---------+-----------+-------------------------------+
| alpine/3.6/arm64 (1 more)            | 9d8005cbf484 | yes    | Alpine 3.6 arm64 (20190619_13:00)            | aarch64 | 3.07MB    | Jun 19, 2019 at 12:00am (UTC) |
+--------------------------------------+--------------+--------+----------------------------------------------+---------+-----------+-------------------------------+
| alpine/3.6/armhf (1 more)            | d4106eea0acf | yes    | Alpine 3.6 armhf (20190619_13:00)            | armv7l  | 3.11MB    | Jun 19, 2019 at 12:00am (UTC) |
+--------------------------------------+--------------+--------+----------------------------------------------+---------+-----------+-------------------------------+
| alpine/3.6/i386 (1 more)             | 52794d2fdb36 | yes    | Alpine 3.6 i386 (20190619_13:00)             | i686    | 3.22MB    | Jun 19, 2019 at 12:00am (UTC) |
+--------------------------------------+--------------+--------+----------------------------------------------+---------+-----------+-------------------------------+
| alpine/3.7 (3 more)                  | 02cd8e69cfe8 | yes    | Alpine 3.7 amd64 (20190619_13:00)            | x86_64  | 3.37MB    | Jun 19, 2019 at 12:00am (UTC) |
+--------------------------------------+--------------+--------+----------------------------------------------+---------+-----------+-------------------------------+
| alpine/3.7/arm64 (1 more)            | 8cd64606e3ea | yes    | Alpine 3.7 arm64 (20190619_13:00)            | aarch64 | 3.25MB    | Jun 19, 2019 at 12:00am (UTC) |
+--------------------------------------+--------------+--------+----------------------------------------------+---------+-----------+-------------------------------+
| alpine/3.7/armhf (1 more)            | 2dd20d3f1bac | yes    | Alpine 3.7 armhf (20190619_13:00)            | armv7l  | 3.28MB    | Jun 19, 2019 at 12:00am (UTC) |
+--------------------------------------+--------------+--------+----------------------------------------------+---------+-----------+-------------------------------+
| alpine/3.7/i386 (1 more)             | dd412e1980d2 | yes    | Alpine 3.7 i386 (20190619_13:00)             | i686    | 3.43MB    | Jun 19, 2019 at 12:00am (UTC) |
+--------------------------------------+--------------+--------+----------------------------------------------+---------+-----------+-------------------------------+
| alpine/3.8 (3 more)                  | 7d7dd381bf08 | yes    | Alpine 3.8 amd64 (20190619_13:00)            | x86_64  | 2.34MB    | Jun 19, 2019 at 12:00am (UTC) |
...
```
この一覧は特定のディストリビューションだけフィルタすることもできる。

例えば、CentOS のイメージだけが見たければ `lxc image list imgases:centos` のようにする

```
$ lxc image list images:centos
+---------------------------+--------------+--------+-----------------------------------+---------+----------+-------------------------------+
|           ALIAS           | FINGERPRINT  | PUBLIC |            DESCRIPTION            |  ARCH   |   SIZE   |          UPLOAD DATE          |
+---------------------------+--------------+--------+-----------------------------------+---------+----------+-------------------------------+
| centos/6 (3 more)         | 52e5cfb4eea3 | yes    | Centos 6 amd64 (20190619_07:08)   | x86_64  | 109.74MB | Jun 19, 2019 at 12:00am (UTC) |
+---------------------------+--------------+--------+-----------------------------------+---------+----------+-------------------------------+
| centos/6/i386 (1 more)    | 3a888cfceae7 | yes    | Centos 6 i386 (20190619_07:08)    | i686    | 109.98MB | Jun 19, 2019 at 12:00am (UTC) |
+---------------------------+--------------+--------+-----------------------------------+---------+----------+-------------------------------+
| centos/7 (3 more)         | 763f3826ffd0 | yes    | Centos 7 amd64 (20190619_07:08)   | x86_64  | 124.90MB | Jun 19, 2019 at 12:00am (UTC) |
+---------------------------+--------------+--------+-----------------------------------+---------+----------+-------------------------------+
| centos/7/arm64 (1 more)   | 6c98f48057fa | yes    | Centos 7 arm64 (20190619_07:08)   | aarch64 | 125.16MB | Jun 19, 2019 at 12:00am (UTC) |
+---------------------------+--------------+--------+-----------------------------------+---------+----------+-------------------------------+
| centos/7/armhf (1 more)   | 927ea26a486b | yes    | Centos 7 armhf (20190619_07:08)   | armv7l  | 122.52MB | Jun 19, 2019 at 12:00am (UTC) |
+---------------------------+--------------+--------+-----------------------------------+---------+----------+-------------------------------+
| centos/7/i386 (1 more)    | 5e3043b4ce2f | yes    | Centos 7 i386 (20190619_07:08)    | i686    | 125.32MB | Jun 19, 2019 at 12:00am (UTC) |
+---------------------------+--------------+--------+-----------------------------------+---------+----------+-------------------------------+
| centos/7/ppc64el (1 more) | 0f24e5285659 | yes    | Centos 7 ppc64el (20190619_07:08) | ppc64le | 126.76MB | Jun 19, 2019 at 12:00am (UTC) |
+---------------------------+--------------+--------+-----------------------------------+---------+----------+-------------------------------+
$
```

### コンテナを作る
起動するイメージが決まったら、ALIAS のところに書かれている名前を使ってコンテナを作る。

コマンドは `lxc launch <イメージ名> <コンテナ名>`。
例えば CentOS7 を作るときは `lxc launch images:centos/7 \<コンテナ名>` のように指定する。
ここではコンテナの名前は `cent7-0` とする。

```
$ lxc launch images:centos/7 cent7-0
Creating cent7-0
Starting cent7-0
$
```
コンテナのイメージはインターネットに公開されているものを取得するため、そこそこ時間がかかる。
"Starting ..." が出ればコンテナは実行されている状態になる。

### コンテナ上でコマンドを実行する
起動したコンテナ上でコマンドを実行するには "lxc exec" コマンドを使う。
exec は `lxc exec \<コンテナ名> \<コマンド> [オプション...]` のように使う。

例えば、 "ip address show" コマンドを作ったコンテナy像で実行するには以下のようにする。
```
$ lxc exec cent7-0 ip a s
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
8: eth0@if9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 00:16:3e:11:1a:80 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.100.111.226/24 brd 10.100.111.255 scope global dynamic eth0
       valid_lft 3420sec preferred_lft 3420sec
    inet6 fd42:b5e9:3e26:b888:216:3eff:fe11:1a80/64 scope global mngtmpaddr dynamic
       valid_lft 3488sec preferred_lft 3488sec
    inet6 fe80::216:3eff:fe11:1a80/64 scope link
       valid_lft forever preferred_lft forever
$
```

実行するコマンドに bash などのシェルを command に指定してもいい。
```
$lxc exec cent7-0 bash
[root@cent7-0 ~]#
```
コマンドを複数実行するときとかはこれが便利。

### コンテナを停止する
実行するプログラムがなくなったときなどは ` lxc stop <コンテナ名>` でコンテナを止めることができる。
一度止めたコンテナは、保存したファイルなどの情報はそのままにコマンドやプログラムが実行されていない状態になる。
これは、通常のOSの電源 off をした時と同じような状態。
```
$ lxc stop cent7-0
To start your first container, try: lxc launch ubuntu:18.04

$
```

このままだとイメージは残ったままでまた起動すれば再開する。
## 備忘録
 - `lxc image export <???>` を使えばコンテナからイメージを作ることができる？
    - そのうち確認する
 - 完全に消したければ `lxc delete <コンテナ名>`　とする。

# 参考にしたサイト
 - https://www.hiroom2.com/2018/12/14/fedora-29-lxd-en/
 - https://linuxcontainers.org/ja/lxd/getting-started-cli/
 - https://linuxcontainers.org/ja/lxd/getting-started-cli/
