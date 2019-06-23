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

### コンテナを作ってに http サーバを立てる
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
TBA

### コンテナ上に http サーバを立てる
(CentOS7 の場合) パッケージのリストに nginx がないので入れるところからやる
```
$ lxc launch images:centos/7

```

