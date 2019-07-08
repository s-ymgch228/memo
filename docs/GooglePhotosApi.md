# Google Photos library APIs を使って写真をアップロードする

手順
1. GoogleAPIs のうち、Google Photos APIs を有効にする
   - ここで client_id, client_secret が手に入る
1. Autorization Code を手に入れる
   - `https://accounts.google.com/o/oauth2/v2/auth` にパラメタをつなげて**ブラウザから**アクセスする
   - 使うパラメタは client_id, scope, redirect_uri
      - scope, redirect_uri は固定値
1. Access Code を入手する

## GoogleAPIs を有効にする
[Get started with RESET](https://developers.google.com/photos/library/guides/get-started?refresh=1)

Google APIs は Google Could Platform (GCP)の一部分という扱い(らしい)
GCP はプロジェクト単位で管理しているため Google Photos APIs を使うためには以下の手順が必要

1. Project を作る
1. プロジェクトの設定で Google APIs を有効にする

ただこれは、Get started の 「ENABLE THE GOOGLE PHOTOS LIBRARY API」を
進めば自動的に設定されるはずなのであまり気にすることはない。

適当なスクリプトから呼び出す場合。Application Type は Other にする。

これで 「Client ID(?)」 と「SECRET」 が手に入る

## Access Token を手に入れる
スクリプトから API をたたくには、OAuth 2.0 を完了させて Access Token を手に入れる必要がある
OAuth2 で Access Token を手に入れるには 「Client ID」、「SECRET」、「Redirect URI」、「SCOPE」が必要。

Client id と SECRET は API 有効時に Application Type を Other にした段階で手に入る。

Redirect URI は `urn:ietf:wg:oauth:2.0:oob` 固定。

> urn:ietf:wg:oauth:2.0:oob  
> この値は、Google の承認サーバーが承認コードをブラウザのタイトル バーに返すことを指定します。

ちなみに、Redirect URI は認証後に遷移するページで、Web Application にした場合にユーザ自身が指定する。
指定した値と、Redirect URI の値は完全一致する必要がある。

SCOPE は [Authetication-authorization](https://developers.google.com/photos/library/guides/authentication-authorization) あたりを見る。

例えば今なら、 write は
```
https://www.googleapis.com/auth/photoslibrary.appendonly
```
共有する API は別(`https://www.googleapis.com/auth/photoslibrary.sharing`)にあって、同時に writeと share を指定したい場合の SCOPE は
この2つを ' '(スペース)をエンコードした "%20" で連結する。

あとはこれを `https://accounts.google.com/o/oauth2/v2/auth` に投げつける。
このリンクの出自は不明
```
URL="https://accounts.google.com/o/oauth2/v2/auth"
clientid=" ... " # 入手しているやつを埋める
ruri="urn:ietf:wg:oauth:2.0:oob"
scope="https://www.googleapis.com/auth/photoslibrary.appendonly"

echo ${URL}?response_type=code&client_id=${clientid}&redirect_uri=${ruri}&scope=${scope}&access_type=offline
```
ここで表示される URL をブラウザにコピペする。リンクを開くと google の画面が出てくるのでログインして最後まで進める。

おそらく最後まで行くと ACCESS TOKEN が手に入るはず

Reference
- - -
- https://developers.google.com/search/apis/indexing-api/v3/get-token?hl=ja
- https://qiita.com/zaki-lknr/items/97c363c12ede4c1f25d2
- https://developers.google.com/adwords/api/docs/guides/authentication?hl=ja
