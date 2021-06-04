# Firebase とは
2014年に Google に買収され，GCP (Google Cloud Platform) に加わった．Google の提供するモバイルおよびWebアプリケーションのバックエンドサービスで，クラウドサービスとしては mBaaS (mobile Backend as a Service) に位置づけされる．開発者がバックエンドのことをあまり気にせずに開発することができる（サーバーレス）．

無料プランと従量制プランがあり，一部機能(Cloud Functions)は従量制でないと使えない．しかし，従量制プランも無料枠があり，その範囲を超えない限り請求はない．

## 機能
### Firebase Realtime Database
クラウドホスト型のNoSQLデータベース．データはJSON形式で保存され，あらゆるデバイスとリアルタイムで同期される．
### Firebase Analystics
アプリでのユーザー行動をデータ化し分析できる機能.
### Firebase Hosting
静的サイトや動的なWebアプリをホスティングできる．
### Firebase Authentication
ユーザーのログインや初期登録の認証システムを簡単に実装できる機能．Google，Twitter や Facebook，GitHub のSNSアカウント認証だけでなく，メールアドレスとパスワードの組み合わせや電話番号認証，匿名認証なども使える．
### Firebase Crashlytics
アプリのクラッシュの検知とその原因の調査に役立つ機能．
### Cloud Functions for Firebase
サーバ不要でバックエンドを開発できる機能．Realtime Database でのデータの変更や新規ユーザーの登録，アナリティクスでのコンバージョンイベントなどに応じて，バックエンドのコードを自動化するといった使い方ができる．
### Cloud Storage for Firebase
ユーザーが作成したコンテンツ（写真や動画など）を保存したり、提供できる機能．バックエンドサービスを自前で開発することなく，ユーザコンテンツを外部に保存するといった使い方ができる．

# 実習

## Node.js のインストール
<https://nodejs.org/ja/>  
```bash
$ node -v
```  
```bash
$ npm -v
```

## Firebase CLI のインストール
```bash
$ npm install -g firebase-tools
```
```bash
$ firebase --version
```

## Firebase プロジェクトの作成
- <https://console.firebase.google.com/>へアクセスしてログイン
- プロジェクトを追加をクリック
- プロジェクト名を適当に決める
- アナリティクスは今回はオフで良い

次に，ターミナルに戻って以下のコマンドを打つ．
```bash
$ firebase login
```
ブラウザが開きログインを求められるので先ほどログインしたアカウントを選択する．  
<br>
次に，プロジェクト用のディレクトリを作成し，そこへ移動して以下のコマンドを打ってFirebaseプロジェクトの初期化を行う．

```bash
$ mkdir YourFirebaseChatApp
$ cd YourFireBaseChatApp
$ firebase init

? Which Firebase CLI features do you want to setup for this folder?
# => Database, Functions, Hosting を選択

? Please select an option:
# => Use an existing project を選択
? Select a default Firebase project for this directory:
# => 先ほどブラウザで作ったプロジェクト名を選択

# 他はエンターを押せばよい
```


## ファイル構成
📂  
┣📂function  
┃┗ 📜index.js  
┣ 📂public  
┃┣ 📜404.html  
┃┣ 📜index.html  
┃┣ 📜main.css <span style="color: #00FF0F; ">(+)</span>  
┃┗ 📜main.js <span style="color: #00FF0F; ">(+)</span>  
┣ 📜database.rules.json  
┗ 📜firebase.json  

## Hosting 側
publicフォルダがホスティングの際に公開される．
### ```public/index.html```
以下に書き換え．
``` html
<html>
<head>
  <meta charset="utf-8">
  <title>Firebase Simple BBS</title>
  <script defer src="/__/firebase/8.6.4/firebase-app.js"></script>
  <script defer src="/__/firebase/8.6.4/firebase-database.js"></script>
  <script defer src="/__/firebase/init.js"></script>
  <script src="./main.js"></script>
  <link rel="stylesheet" type="text/css" href="./main.css"></link>
</head>
<body>
  <div id="reply-form">
    <div class="l">Name:</div> <input type="text" id="reply-name">
    <div class="l">Body:</div> <textarea id="reply-body"></textarea>
    <div class="l">Key:</div> <input type="text" id="reply-key">
    <div class="l"></div> <button id="reply-submit" onclick="postReply()">Post</button>
  </div>
  <div id="replies"></div>
</body>
</html>
```  

### ```public/main.js```
新規作成．
``` js
const main = ()=>
  firebase.database().ref('/simplebbs/posts').limitToLast(10).on('value', snapshot=>{
    const posts = snapshot.exists() ? snapshot.val() : {}
    let html = ''
    for(const [id, {name, content, date}] of Object.entries(posts).reverse())
      html += makeReply(id, name, content, date)
    document.querySelector('#replies').innerHTML = html
  })

const makeReply = (id, name, content, date) => `<div class="reply">
  <div class="head">Name: ${name} <span class="date">${date}</span></div>
  <div class="content">${content}</div>
  <button class="delete" onclick="deleteReply('${id}')">delete</button> </div>`

const postReply = ()=> post('/api/post', {
    name: document.querySelector('#reply-name').value,
    content: document.querySelector('#reply-body').value,
    key: document.querySelector('#reply-key').value,
  }).then(e=>{ document.querySelector('#reply-body').value='' })

const deleteReply = id => post('/api/delete', {id, key: prompt('key?') || ''})

const post = (path, jsonData) => fetch(path, {
    method: 'POST', body: JSON.stringify(jsonData),
    headers: {'Accept': 'application/json', 'Content-Type': 'application/json'},
  })

document.addEventListener('DOMContentLoaded', main)
```

### ```public/main.css```
新規作成．
``` css
body {background: #ECEFF1; color: #757575;}
#reply-form, #replies {margin: 100px auto 16px; max-width: 360px;}
#reply-form, .reply {background: white; padding:5px;}
.reply {position: relative; border: 1px solid #ddd; margin:10px;}
#reply-body {width: 100%; height: 7em;}
#reply-key {width: 50px;}
.head {color: #888; font-size: 75%;}
.date {position: absolute; right: 0;}
.content {border-top: 1px dotted #ddd; padding: 4px;}
.delete {position: absolute; bottom: 0; right: 0; font-size: 60%; padding: 1px; background-color: #f5f5f5; border: none; color: #ccc; opacity: 0.7;}
.l {font-size: .8rem;}
```

## Function 側

### ```functions/index.js```
以下に書き換え．
``` js
const functions = require('firebase-functions')
const admin = require('firebase-admin')
admin.initializeApp(functions.config().firebase)
const db = admin.database()
const rep = s=> s.replace(/</g, '&lt;').replace(/>/g, '&gt;').replace(/\n/g,'<br>')

exports.post = functions.https.onRequest((request, response)=>{
  const {name, content, key} = request.body
  const date = new Date().toLocaleDateString()
  db.ref('/simplebbs/posts').push({name:rep(name), content:rep(content), date})
    .then(e=> db.ref(`/simplebbs/keys/${e.key}`).set(key))
    .then(e=> response.status(200).end())
})

exports.delete = functions.https.onRequest((request, response)=>{
  const {id, key} = request.body
  db.ref(`/simplebbs/keys/${id}`).once('value').then(sKey=>{
    if(!sKey.exists() || sKey.val() !== key)
      return response.status(400).send('invalid id or incorrect key').end()
    db.ref(`/simplebbs/posts/${id}`).remove()
      .then(e=> sKey.ref.remove())
      .then(e=> response.status(200).end())
  })
})
```
## 設定ファイル等
### ```firebase.json```
以下に書き換え．
```json
{
  "database": {
    "rules": "database.rules.json"
  },
  "hosting": {
    "public": "public",
    "ignore": [
      "firebase.json",
      "**/.*",
      "**/node_modules/**"
    ],
    "rewrites": [
      {"source": "/api/post", "function": "post"},
      {"source": "/api/delete", "function": "delete"}
    ]
  }
}
```

### ```database.rules.json```
以下に書き換え．
``` json
{
  "rules": {
    ".read": "false",
    ".write": "false",

    "simplebbs":{
      "posts":{
        ".read": "true"
      }
    }
  }
}
```

## デプロイ
ターミナルで以下のコマンドを打つ．
```bash
$ firebase serve 
# => ローカルでテスト
```
もしくは
```bash
$ firebase deploy
# => デプロイ
```
おそらく，ここでデプロイしても無料プランだと Cloud Function が使えないのでエラーが出る．  
Webで動いているのがみたい人はエラーで出力されたURLに飛び従量制プランへ変更してもう一度上記のコマンドを叩いてほしい．プランはアカウントごとではなくプロジェクトごとに管理される．従量制プランでも無料容量以内なら請求はない．心配な場合は本実習後にプロジェクトの停止または削除を行う（後述）．  
<br>
完了すると出力されるホスティングURLで正しく動作しているか確認する．  
違うブラウザで二つウィンドウを並べたり，異なるデバイスの二つの画面で動作を確認すると，リアルタイムで同期されていることが良くわかる．

## プロジェクトの停止または削除方法
### 停止方法
```bash
$ firebase hosting:disable
```
デプロイしたサイトが停止する．

### 削除方法
![プロジェクトの削除方法](https://github.com/hinapupil/semi0604/blob/main/Animation.gif)