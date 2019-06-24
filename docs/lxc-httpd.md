# lxc 上に nginx で httpd を立てる
ホスト上に http で公開するファイルを置く。そのファイルを lxc 上で参照できるようにして httpd で公開する。

```
+-----------+
|   httpd   |
+-----------+ +---------------------------+
| container | | host のプログラム(catとか) |
+-----------+-+---------------------------+
|      host OS (ここにファイルが保存される) |
+-----------------------------------------+
```
ホストに保存されてる適当なファイルをコンテナ上の httpd で公開する

### 適当なコンテナを作る
(CentOS7 の場合) パッケージのリストに nginx がないので入れるところからやる
```
$ lxc launch images:centos/7 httpd0
Creating httpd0
Starting httpd0 
$ lxc exec httpd0 bash
[root@httpd0 ~]# cat > /etc/yum.repo.d/nginx.repo << _EOF
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

### ホストとコンテナ間でディレクトリを共有する
ホストと共有するディレクトリはコンテナ側から見るとデバイスとして追加される。
利用するコマンドは "lxc config device add"　で、以下のような形式をとる。
```
lxc config device add \<コンテナ名> \<デバイス名> \<デバイスのタイプ> soure=/path/to/hostdir path=/path/to/container/dir
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

