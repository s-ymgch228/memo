# Google Photos library APIs を使って写真をアップロードする
   
1. OAuth 2.0 の認証を行って client_id, client_secret, refresh_token を入手する
1. client_id, client_secret, refresh_token から、Access Token を手に入れる
1. Access Token を使って API にアクセスする

## OAuth2 の Access Token を入手する
やり方は [Google API OAuth2.0のトークン取得手順](https://qiita.com/giiko_/items/b0b2ff41dfb0a62d628b) にある通り。

### client id　/ client secret を作る
[Get start with REST](https://developers.google.com/photos/library/guides/get-started) の "enable API" を行って
「client id」と「client secret」 をつくる。この値は管理画面から見れるので特に保存する必要はない。
shell script で動かす場合でも Application type は Web Applicaton としておく。redirect url は `http://localhost` にする

### scope を決める
scope は URL の形をしていて
[Authentication & authorization](https://developers.google.com/photos/library/guides/authentication-authorization) 
にあるリストから用途に合ったものを選択する。

複数の scope を使いたい場合は "%20" で `SCOPE_0%02%SCOPE_1%20...` のようにつなげる。

### authorization code を手に入れる
**web ブラウザ**を使って `https://accounts.google.com/o/oauth2/auth`へアクセスする
リンクはパラメタをつなげて以下のようにする。

```
baseurl="https://accounts.google.com/o/oauth2/auth"
client_id="hidden"
client_secret="hidden"
redirect_uri="http://localhost"
scope="https://www.googleapis.com/auth/photoslibrary"

cat << _EOF
${baseurl}\
?client_id=${client_id}\
&redirect_uri=${redirect_uri}\
&scope=${scope}\
&response_type=code\
&approval_prompt=force\
&access_type=offline
_EOF
```
このリンクを踏むと、Google の認証ページに飛ばされる。
「途中認証されていないアプリ」みたいな警告が出てくるが、危険を理解して... 的なところから認証を進める。
認証が完了したら、接続できないページが出てくる。
このページのリンクの `code=` がauthorization code になる。

authorization code が手に入るとそこから access token と refresh_token が取得できる
```
client_id="hidden"
client_secret="hidden"
redirect_uri="http://localhost"

code="hidden"
curl \
-d "client_id=${client_id}" \
-d "client_secret=${client_secret}" \
-d "redirect_uri=${redirect_uri}" \
-d "grant_type=authorization_code" \
-d "code=${code}" https://accounts.google.com/o/oauth2/token
```

Access Token は expires_in が設定されていて、（おそらく）取得後1時間しか利用できない。
なので適当なスクリプトで動かすときは
1. refresh

Reference
- - -
1. [Google API OAuth2.0のトークン取得手順](https://qiita.com/giiko_/items/b0b2ff41dfb0a62d628b)
   - これが一番役に立った。
1. [Google OAuth2 Tutorial](https://requests-oauthlib.readthedocs.io/en/latest/examples/google.html)
1. [Google API OAuth2](https://qiita.com/giiko_/items/b0b2ff41dfb0a62d628b)
