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
OAuth2 は 「Client ID」、「SECRET」、「Redirect URI」、「SCOPE」が必要。

Client id と SECRET は API 有効次に Application Type を Other にした段階で手に入る。

Redirect URI は `urn:ietf:wg:oauth:2.0:oob` 固定。

> urn:ietf:wg:oauth:2.0:oob  
> この値は、Google の承認サーバーが承認コードをブラウザのタイトル バーに返すことを指定します。

Redirect URIは、認証後に遷移させる Web Application のアドレスで、
Application Type に Web Application を指定するときに同時に設定する。
このアドレスは事前に設定したものと完全一致しないといけない。

SCOPE は [Authetication-authorization](https://developers.google.com/photos/library/guides/authentication-authorization) あたりを見る。

例えば今なら、 write は
```
https://www.googleapis.com/auth/photoslibrary.appendonly
```
共有する API は別(`https://www.googleapis.com/auth/photoslibrary.sharing`)にあって、同時に writeと share を指定したい場合の SCOPE は
この2つを ' '(スペース)をエンコードした "%20" で連結する。

