# setup
(TBD)
1. bridge を作っておく
   - 外向けのインタフェースをホストとコンテナで共有しつつホストとコンテナ間の通信もできる
2. snap で lxd いれる
3. lxd init
   - いつもやってる設定があったような
   - pool は lvm にしない
     - ホストごと固まるっぽい(?)
   - disk にもしない
     - 性能が出ないらしい(?)  

# ステータス取得系
- `lxc list`
   - 作ったコンテナ一覧
- `lxc config device show <container>`
   - 追加したデバイス一覧 

# コンテナ作る
ベースにするイメージを決める
使えるイメージ一覧：

`lxc image list images:`

CentOS だけみたい時は centos で引っ掛ける

`lxc image list images:centos`

使うイメージを決めたらコンテナを作る

`lxc launch images:<image> <container>`

| パラメタ | 説明 | 例 |
|:-------|:-----|:--|
| \<image\> | images のリストにあるやつを指定する | CentOS/8-Stream |
| \<container\> | コンテナ名 | (任意の文字列) |

tips:
 - CentOS 7 系
   - 停止や再起動できない

# NIC を足す
`lxc config device add <container> <device> nic nictype=bridged parent=<nic>`

| パラメタ | 説明 | 例 |
|:-------|:-----|:--|
| \<container\> | コンテナ名 |
| \<device\> | デバイス名 | 適当な文字列
| \<nic\> | ホスト側の nic を指定する。あらかじめ bridge を作っておいてそれを指定する。 |

# ホストのディレクトリを lxc 側で読み書きできるようにする

`lxc config device add <container> <device> disk source=<hostdir> path=<containerpath>`

| パラメタ | 説明 | 例 |
|:-------|:-----|:--|
| \<container\> | コンテナ名 |
| \<device\> | デバイス名 | 適当な文字列
| \<host-dir\> | ホスト側のパス | 
| \<container-dir\> | コンテナ側のパス |

# コンテナの root をホスト側の適当な uid/gid にマッピングする
```
cat << EOS | lxc config set <container> raw.idmap -
uid <host-uid> <container-uid>
gid <host-gid> <container-gid>
EOS
```
| パラメタ | 説明 | 例 |
|:-------|:-----|:--|
| \<container\> | コンテナ名 |
| \<host-uid\> | ホスト側の uid |
| \<container-uid\> | コンテナ側 の uid | 通常 root の '0' |
| \<host-gid\> | ホスト側の uid |
| \<container-gid\> | コンテナ側の gid |  通常 root の '0' |
