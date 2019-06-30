# lxc のコマンド備忘録
[コンテナ - LXDドキュメント翻訳プロジェクト](https://lxd-ja.readthedocs.io/ja/latest/containers/)

## ディスクデバイスの追加

```
$ lxc config device add <container> <device-name> disk source=path/to/host/dir path=path/to/container/dir
```

例
```
lxc config device add httpd0 http_root disk source=/var/lib/httproot path=/mnt/http_root
```
### 注意
エラーが出ることがある
```
Error: Failed to run: /snap/lxd/current/bin/lxd forkstart gitsrv0 /var/snap/lxd/common/lxd/containers /var/snap/lxd/common/lxd/logs/gitsrv0/lxc.conf:
Try `lxc info --show-log gitsrv0` for more info
```

おそらく↓のバグ
https://github.com/lxc/lxd/issues/5788

## proxy デバイス
ホストのアドレスを特定のコンテナに転送する機能
```
lxc config device add <container> <device-name> proxy listen=<type>:<addr>:<port>[-<port>][,<port>] connect=<type>:<addr>:<port>
```

例
```
$ lxc config device add rproxy0 http proxy listen=tcp:0.0.0.0:80 connect=tcp:127.0.0.1:80
Device http added to rproxy0
```
ホスト側の firewall や selinux の設定によっては転送するように設定していても discard されていることもある
