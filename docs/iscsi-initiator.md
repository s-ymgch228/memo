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

対向の情報を見るには `iscsiadm -m node -o show` を使う
```
# iscsiadm -m node -o show 
# BEGIN RECORD 2.0-876
node.name = iqn.2005-10.org.freenas.ctl:isd0                                                                                           
node.tpgt = 1                                                                                                                          
node.startup = automatic
node.leading_login = No
...
node.conn[0].iscsi.DataDigest = None
node.conn[0].iscsi.IFMarker = No                                                                                                         
node.conn[0].iscsi.OFMarker = No                                                                                                         
# END RECORD  
```
ここに確認する情報は特になくて表示がされていればいいように見える

ここまでくるとあとはログインするだけでディスクが見えるようになる
```
# iscsiadm -m node --login                                                                                                               
Logging in to [iface: default, target: iqn.2005-10.org.freenas.ctl:isd0, portal: 192.168.122.2,3260] (multiple)                         
Login to [iface: default, target: iqn.2005-10.org.freenas.ctl:isd0, portal: 192.168.122.2,3260] successful.   
```

dmesg を見ると iscsi ディスクを認識したことを示すログが出ている
```
...
[778387.878932] Loading iSCSI transport class v2.0-870.
[778387.906989] iscsi: registered transport (tcp)
[936783.392867] scsi host2: iSCSI Initiator over TCP/IP
[936783.410162] scsi 2:0:0:0: Direct-Access     FreeNAS  iSCSI Disk       0123 PQ: 0 ANSI: 7
[936783.411771] sd 2:0:0:0: Attached scsi generic sg1 type 0
[936783.411785] sd 2:0:0:0: Power-on or device reset occurred
[936783.416564] sd 2:0:0:0: [sdb] 1048576000 512-byte logical blocks: (537 GB/500 GiB)
[936783.416567] sd 2:0:0:0: [sdb] 16384-byte physical blocks
[936783.416728] sd 2:0:0:0: [sdb] Write Protect is off
[936783.416730] sd 2:0:0:0: [sdb] Mode Sense: 7f 00 10 08
[936783.417040] sd 2:0:0:0: [sdb] Write cache: enabled, read cache: enabled, supports DPO and FUA
[936783.417428] sd 2:0:0:0: [sdb] Optimal transfer size 1048576 bytes
[936783.425057] sd 2:0:0:0: [sdb] Attached SCSI disk
...
```

本当はログイン名を決めてそれに合わせて見えるディスクを変えたほうがいいはず。

あとは普通に fdisk パーティションを作って mkfs する
```
[root@NASsv0 ~]# fdisk /dev/sdb

Welcome to fdisk (util-linux 2.33.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0x77532e13.

Command (m for help): p
Disk /dev/sdb: 500 GiB, 536870912000 bytes, 1048576000 sectors
Disk model: iSCSI Disk
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 16384 bytes
I/O size (minimum/optimal): 16384 bytes / 1048576 bytes
Disklabel type: dos
Disk identifier: 0x77532e13

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-1048575999, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-1048575999, default 1048575999):

Created a new partition 1 of type 'Linux' and of size 500 GiB.

Command (m for help): t
Selected partition 1
Hex code (type L to list all codes): 83
Changed type of partition 'Linux LVM' to 'Linux'.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
#
```

```
# mkfs.ext4 /dev/sdb1
mke2fs 1.44.6 (5-Mar-2019)
Discarding device blocks: done
Creating filesystem with 131071744 4k blocks and 32768000 inodes
Filesystem UUID: 721d1f03-27c5-437d-bf77-74a685b6fbb1
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968,
        102400000

Allocating group tables: done
Writing inode tables: done
Creating journal (262144 blocks): done
Writing superblocks and filesystem accounting information: done 
```
