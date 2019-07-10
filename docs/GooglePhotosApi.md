# Google Photos library APIs を使って写真をアップロードする
   
1. (初回のみ)OAuth 2.0 の認証を行って client_id, client_secret, refresh_token を入手する
1. client_id, client_secret, refresh_token から、Access Token を手に入れる
1. Access Token を使って API にアクセスする

## OAuth2 の Refresh Token を入手する
Refresh Token は Access Token を初めて取得するときに一緒に手に入る。
Access Token の初回取得は
[Google API OAuth2.0のトークン取得手順](https://qiita.com/giiko_/items/b0b2ff41dfb0a62d628b)
にある通り。

1. APIを有効にして client_id と client secret を手に入れる
1. scope を決めて`https://accounts.google.com/o/oauth2/auth` に接続する
1. 認証に成功すると Authorization code が手に入る
1. authorization code を使って Access token と Refres Token を手に入れる

### やり方
[Get start with REST](https://developers.google.com/photos/library/guides/get-started)
の "enable API" のリンクから「client id」と「client secret」 をつくる。

"enable API" の途中で聞かれる Application type は Web Application を選択する。
Redirect URL は "http://localhost" などのつながらないリンクにしておく

初回の Access Token の入手には scope と呼ばれる URL も必要。
scope は[Authentication & authorization](https://developers.google.com/photos/library/guides/authentication-authorization)
に書かれている。

複数の scope を指定したいときは `SCOPE_0%02%SCOPE_1%20...` みたいに "%20" で連結する。

あとはこれを `https://accounts.google.com/o/oauth2/auth`につなげて **適当な Web ブラウザ**から接続する
認証リクエストの URL に **approval_prompt=force** と **access_type=offline** をつける必要があることに注意が必要。
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
なので適当なスクリプトで動かすときはその都度 refresh_token を使って Access Token を手に入れる必要がある

## APIにアクセスするスクリプトを作る
Access Token はJSON 形式の文字列で送られてくるため JSON を解析できる ruby などの適当な言語を使うほうがいい。

```ruby
#!/usr/bin/ruby
require 'net/http'
require 'uri'
require 'json'

$debug=true

REFRESH_TOKEN = "hidden"
CLIENT_ID = "hidden"
CLIENT_SECRET = "hidden"



begin
  uri=URI.parse('https://www.googleapis.com/oauth2/v4/token')
  query = {
    "refresh_token" => REFRESH_TOKEN,
    "client_id"     => CLIENT_ID,
    "client_secret" => CLIENT_SECRET,
    "grant_type"    => "refresh_token"
  }

  res = Net::HTTP.post_form(uri, query)
  token = JSON.parse(res.body) if res.code == "200"
rescue => e
  puts "#{e}"
end

exit 1 if token.nil?

puts "token=#{token}" if $debug
```

Reference
- - -
1. [Google API OAuth2.0のトークン取得手順](https://qiita.com/giiko_/items/b0b2ff41dfb0a62d628b)
   - これが一番役に立った。
1. [Google OAuth2 Tutorial](https://requests-oauthlib.readthedocs.io/en/latest/examples/google.html)
1. [Google API OAuth2](https://qiita.com/giiko_/items/b0b2ff41dfb0a62d628b)
