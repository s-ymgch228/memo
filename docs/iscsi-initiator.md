# Fedora 30 で iSCSI ディスクに接続する

1. 対抗は FreeNAS11
   - FreeNAS 上に iSCSI を構成する方法は適当に探せば出てくる

## FreeNAS の設定
探せば像付きで出てくるのでここではあっさりと書いておく。

1. zvol を適当なストレージに作成する
   - ここで設定するディスクのサイズが iSCSI ディスクのサイズになる
1. (iSCSI の) Target Global Configuration にベースネームを設定する
   - FQDN ならなんでもいい？
1. Portal を追加する
   - Discovery Auth Method : なし
   - 認証グループの探索 : なし
   - コメント : 任意
   - IP アドレス : 任意
1. イニシエーターを追加する
   - 接続元（クライアント）のアドレス/ネットワークアドレスを設定する
   - 特に制限なしなら ALL/ALL
1. ターゲット
   - ポータルグループID, イニシエータグループIDはこの前の手順で作ったものを指定する
1. エクステンド
   - 名前 : 任意につける
   - エクステントタイプ: zvol を公開する場合はデバイス、datasets を作った場合はディレクトリを選択する
   - デバイス (or パス) : 作ったzvol のパス
1. Asosiated Targets
   - エクステンド名を選択する

## Fedora 30 から接続する
iscsi-initiator-utils を入れる（初めから入ってるかも？）
```
% sudo dnf install -y iscsi-initiator-utils
```

scsi target を探す
```
 #  iscsiadm -m discovery -t sendtargets -p 192.168.122.2
192.168.122.2:3260,1 iqn.2005-10.org.freenas.ctl:isd0
```
このコマンドの出力内容:
- 192.168.122.2:3260 は NAS サーバの IP アドレス/ポート番号
- iqn.2005-10.org.freenas.ctl は Target GlobalConfiguration に指定したFQDN
- isd0 はターゲットに指定した値
