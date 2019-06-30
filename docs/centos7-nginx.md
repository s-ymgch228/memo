# lxc で centos/7 のイメージを作って nginx を立ち上げる

## 適当なコンテナ(rproxy0)を作る
```
$ lxc launch images:centos/7 rproxy0
Creating httpd0
Starting httpd0 
$ lxc exec rproxy0 bash
[root@rproxy0 ~]# cat > /etc/yum.repos.d/nginx.repo << _EOF
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/mainline/centos/7/\$basearch/
gpgcheck=0
enabled=1
_EOF
[root@rproxy0 ~]# yum install -y nginx                                                                                                   
Failed to set locale, defaulting to C
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: ftp.iij.ad.jp
 * extras: ftp.iij.ad.jp
 * updates: ftp.iij.ad.jp
Resolving Dependencies
--> Running transaction check
---> Package nginx.x86_64 1:1.17.1-1.el7.ngx will be installed
--> Processing Dependency: openssl >= 1.0.2 for package: 1:nginx-1.17.1-1.el7.ngx.x86_64
--> Running transaction check
---> Package openssl.x86_64 1:1.0.2k-16.el7_6.1 will be installed
--> Processing Dependency: make for package: 1:openssl-1.0.2k-16.el7_6.1.x86_64
--> Running transaction check
---> Package make.x86_64 1:3.82-23.el7 will be installed
--> Finished Dependency Resolution

...

Installed:
  nginx.x86_64 1:1.17.1-1.el7.ngx

Dependency Installed:
  make.x86_64 1:3.82-23.el7                                                openssl.x86_64 1:1.0.2k-16.el7_6.1

Complete!
[root@rproxy0 ~]#
```
あとは普通に nginx を起動しておく

```
[root@rproxy0 ~]# systemctl status nginx
● nginx.service - nginx - high performance web server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
     Docs: http://nginx.org/en/docs/
[root@rproxy0 ~]# systemctl enable nginx
Created symlink from /etc/systemd/system/multi-user.target.wants/nginx.service to /usr/lib/systemd/system/nginx.service.
[root@rproxy0 ~]# systemctl start nginx
[root@rproxy0 ~]#
```

## コンテナに proxy デバイスを追加する
`lxc config device add <コンテナ名> <デバイス名> proxy` でホストの特定のポートを転送する
Note: [コンテナ - LXDドキュメント翻訳プロジェクト](https://lxd-ja.readthedocs.io/ja/latest/containers/)

コマンドのフォーマット:
```
lxc config device add <container> <device-name> proxy listen=<type>:<addr>:<port>[-<port>][,<port>] connect=<type>:<addr>:<port>
```

これを **ホスト側** で実行する
```
$ lxc config device add rproxy0 http proxy listen=tcp:0.0.0.0:80 connect=tcp:127.0.0.1:80
Device http added to rproxy0
$
```
コンテナ側のアドレスは `127.0.0.1` にしている。これは、コンテナに設定されているアドレスでもいい。
ここではコンテナに明示的にアドレスを割り当ててなくて動的に割り当ててあるのでアドレス変更があっても影響しないローカルホストのアドレスにした。
