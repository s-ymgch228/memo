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

おそらく https://github.com/lxc/lxd/issues/5788 のバグ

#### lxc config device add するとエラーが出る。
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

## NIC 追加
物理ネットワークに接続したいときは nic を追加する。

```
lxc config device add flashair0 phy-nic nic nictype=bridged parent=br0
```
flashair0 はコンテナ名、その次の phy-nic は適当につけたデバイス名

~~nictype はいくつかあるうちの macvlan を選んだ
macvlan は物理NICからパケットを送受信する場合にブリッジより効率がいい（らしい）。~~

あらかじめホスト側に Bridge インタフェースを nmcli などで作るようにしておいて nictype は bridged にする。
macvlan を選ぶと container 側の nic も macvlan になり、NetworkManager がエラーをはいて dhcp client が自動起動しないことがある。

macvlan に似たやつに ipvlan があるがこっちは物理 NIC の MAC アドレスを使いまわす。そのため dhcp とは相性が悪そう。
その一方で MAC アドレスの総数が増えないので L2SW 的には負荷が少なそう

### ネットワークの追加
`<lxdbrN>` は作りたい名前に置き換える
```
lxc network create <lxdbrN> bridge.external_interfaces=enp3s0 ipv4.nat=true
```

パラメタ
- bridge.external_interfaces=enp3s0
   - 外部のNIC
- ipv4.nat
   - NATするかどうか

bridge.external_interfaces するときは lxd 専用の NIC が必要らしい(?)

### 固定 IP
```
$ lxc network attach lxdbr0 <コンテナ名> eth0 eth0
$ lxc config device set <コンテナ名> eth0 ipv4.address <IPアドレス>
$ lxc restart <コンテナ名>
```
network attach はすでに eth0 がある場合でも必要。
この後にコンテナ側で `/etc/sysconfig/network-config/ifcfg-eth0` などを編集して固定する。
