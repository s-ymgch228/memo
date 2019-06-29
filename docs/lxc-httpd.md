# lxc 上に nginx で httpd を立てる
lxc で web アプリを立ち上げると異なるアドレス/同じポート番号でプロセスを立ち上げることが多い
それをリモートのクライアントから接続する場合は

1. ホストの特定のポートに来た通信をコンテナに転送する、NAPT のようなことをする
1. ホストに複数の IP アドレスを割り当てておいてコンテナのアドレスに変換(NAT) する
1. コンテナを外部のネットワークからアクセスできるようにする

のいずれかコンテナとの疎通性があるようにする必要がある。
ここでは、特定のポートをコンテナに転送する proxy の利用方法をまとめる。

## 構成

```
+---------+     +---------+     +----+----+
| rproxy0 |     | webapp0 |     | webapp1 |
+----+----+     +----+----+     +----+----+
     |               |               |
-----+-------+-------+---------------+----- container NW
             |
+------------+-----------------------+
|          host os                   |
+------------------------------------+
```

1. http のリバースプロキシ用コンテナを作る
1. host os の 80/tcp をリバースプロキシへ転送する
1. リバースプロキシはそのままウェブアプリのコンテナに転送する

## 適当なコンテナを作る
(CentOS7 の場合) パッケージのリストに nginx がないので入れるところからやる
```
$ lxc launch images:centos/7 httpd0
Creating httpd0
Starting httpd0 
$ lxc exec httpd0 bash
[root@httpd0 ~]# cat > /etc/yum.repos.d/nginx.repo << _EOF
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/mainline/centos/7/\$basearch/
gpgcheck=0
enabled=1
_EOF
[root@httpd0 ~]# yum search nginx
...
nginx.x86_64 : High performance web server
...
[root@httpd0 ~]# yum install -y nginx
...
* http://nginx.com/products/

----------------------------------------------------------------------
    1/3   Verifying  : 1:nginx-1.17.0-1.el7.ngx.x86_64
    2/3   Verifying  : 1:openssl-1.0.2k-16.el7_6.1.x86_64
    3/3   Verifying  : 1:make-3.82-23.el7.x86_64

Installed:
  nginx.x86_64 1:1.17.0-1.el7.ngx
Dependency Installed:
  make.x86_64 1:3.82-23.el7
  openssl.x86_64 1:1.0.2k-16.el7_6.1

Complete!
[root@httpd0 ~]#
```

## ホストとコンテナ間でディレクトリを共有する
ホストと共有するディレクトリはコンテナ側から見るとデバイスとして追加される。
利用するコマンドは "lxc config device add"　で、以下のような形式をとる。
```
lxc config device add <コンテナ名> <デバイス名> <デバイスのタイプ> soure=/path/to/hostdir path=/path/to/container/dir
```
\<コンテナ名> は作ったコンテナの名前、\<デバイス名> は追加するデバイスの取り外しなどで指定する適当な名前を指定する。
source, path はぞれぞれホストのパスとコンテナ側のパスを指定する

Note: [コンテナ - LXDドキュメント翻訳プロジェクト](https://lxd-ja.readthedocs.io/ja/latest/containers/)

```
lxc config device add httpd0 http_root disk source=/var/lib/httproot path=/mnt/http_root
```


TBA

### コンテナ上に http サーバを立てる
(CentOS7 の場合) パッケージのリストに nginx がないので入れるところからやる
```
$ lxc launch images:centos/7

```

- - -
はまったところ
## 1. ホスト側で nfs マウントしたディレクトリはコンテナ側に disk デバイスとして追加できない？
lxc config device add するとエラーが出る。
```
$ sudo mount -t nfs nfsserver:/mnt/path /mnt
$ lxc launch images:centos/7 nfsc0
Creating nfsc0
Starting nfsc0
$ lxc config device add nfsc0 httprt disk source=/mnt path=/mnt
Error: Add disk devices: Failed to add mount for device: Failed to run: /snap/lxd/current/bin/lxd forkmount lxc-mount nfsc0 /var/snap/lxd/common/lxd/containers /var/snap/lxd/common/lxd/logs/nfsc0/lxc.conf /var/snap/lxd/common/lxd/devices/nfsc0/disk.httprt.mnt /mnt none 4096:                                       
$ lxc config device add nfsc0 httprt disk source=/mnt/nfsshare/ path=/mnt
Error: Add disk devices: Failed to setup device: remove /var/snap/lxd/common/lxd/devices/nfsc0/disk.httprt.mnt: device or resource busy 
$
```
なお、一度この状態になるとコンテナを削除して**同じコンテナ名で**作り直してもエラーが取れない。
```
$ lxc launch images:centos/7 nfsc0
Creating nfsc0
Starting nfsc0
Error: Failed to run: /snap/lxd/current/bin/lxd forkstart nfsc0 /var/snap/lxd/common/lxd/containers /var/snap/lxd/common/lxd/logs/nfsc0/lxc.conf:
Try `lxc info --show-log local:nfsc0` for more info
$ lxc info --show-log local:nfsc0
...
lxc nfsc0 20190626134458.413 ERROR    utils - utils.c:safe_mount:1187 - Invalid argument - Failed to mount "/var/snap/lxd/common/lxd/shmounts/nfsc0" onto "/$ar/snap/lxd/common/lxc//dev/.lxd-mounts"
lxc nfsc0 20190626134458.414 ERROR    conf - conf.c:mount_entry:2044 - Invalid argument - Failed to mount "/var/snap/lxd/common/lxd/shmounts/nfsc0" on "/var$snap/lxd/common/lxc//dev/.lxd-mounts"
...
```
この状態から回復させるにはホスト側の lxd.daemon.service を再起動する必要がある
```
$ sudo systemctl restart snap.lxd.daemon
```
