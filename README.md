# Denoでしりとり
## 目次
1. 環境構築手順
2. しりとりの実装
3. 実装機能
4. ルールの概要
5. 改良点
### 1. 環境構築手順
#### 1.1. Denoをインストール
下記サイトから自分が持っているOSに合わせてインストールする。
```
https://docs.deno.com/runtime/manual
```
#### 1.2. Visual Studio Codeをインストールする
下記サイトから自分が持っているOSに合わせてインストールする。
```
https://code.visualstudio.com/
```
#### 1.3. Visual Studio Code内のターミナルで実行
##### 1.3.1. Denoのバージョンを調べる
ターミナルで以下のコマンドを実行する。
```
deno --version
```
そうすると
```
deno 1.44.0 (release, x86_64-apple-darwin)
v8 12.6.228.3
typescript 5.4.5
```
と表示される。  
##### 1.3.2. Denoで「Hello World!」
server.jsを作成する。  
server.jsで実行するためにソースコードを以下に示す。  
```
console.log("Hello World!");
```
実行するコマンドを以下に示す。
```
deno run server.js
```
Hello Worldと出力されたら、正常にプログラムが動いているということである。  

#### 1.4. 拡張機能Denoをインストールする
画面左側の「Extensions」アイコンから拡張機能の画面を開き、「Deno」で検索し、インストールする。  
##### 1.4.1. Denoの拡張機能の設定
server.jsを作成したフォルダで、Denoの拡張機能の設定を行う。  
「Control+Shift+P」でコマンドパレットを開き、「Deno: Initialize Workspace Configuration」を実行する。   
(※このときにワークスペースを作成しないと実行できないようになるので確認しておく。)  
.vscode/settings.jsonが作成され、Deno関連の補完が効くようになる。
#### 1.5. DenoでHTTPサーバーを立ち上げる
server.jsの内容を以下のように書き換えて保存する。
```
// server.js

// localhostにDenoのHTTPサーバーを展開
Deno.serve(request => {
    return new Response("Hello Deno!");
});
```
--allow-netオプションをつけてserver.jsを起動する。
```
deno run --allow-net server.js
```
ブラウザで<http://localhost:8000>にアクセスする。  

ブラウザにHello Deno!と表示されれば、プログラムは正常に動いている。
#### 1.6 サーバーに変数を定義する
server.jsファイルを以下の内容に書き換える。
```
// server.js
 // アクセス数を保持する変数をグローバル領域に定義
 let count = 0;
 
  // localhostにDenoのHTTPサーバーを展開
  Deno.serve(request => {
     count++;
     return new Response(`Count: ${count}`);
  });
```
--allow-netに加えて、--watchオプションを追加してserver.jsを起動する。  
```
deno run --allow-net --watch server.js
```
このオプションを指定すると、Denoがファイルの変更を自動でサーバーに反映してくれる。  

ブラウザで<http://localhost:8000>にアクセスする。  

何回かアクセスして、ブラウザにアクセス回数が表示されれば、プログラムは正常に動いている。  
尚、ブラウザが自動で/favicon.icoを取得しようとするため、カウントが2つずつカウントアップすることがある。  
#### 1.7 HTMLを表示させてみる
server.jsファイルを以下の内容に書き換える。
```
  // localhostにDenoのHTTPサーバーを展開
  Deno.serve(request => {

     return new Response(
         // Responseの第一引数にレスポンスのbodyを設置
         "<h1>H1見出しです</h1>",
         // Responseの第二引数にヘッダ情報等の付加情報を設置
         {
             // レスポンスにヘッダ情報を付加
             headers: {
                 // text/html形式のデータで、文字コードはUTF-8であること
                 "Content-Type": "text/html; charset=utf-8"
             }
         }
     );
  });
```
ブラウザを再読み込みして、「H1見出しです」と大きく表示される。
#### 1.8 ファイルサーバーを実装してみる
##### 1.8.1. HTMLファイルを読み込んでみる
public-1フォルダを作成し、中にindex.htmlを作成する。フォルダ構成は以下のようになる。
```
├─ .vscode/
├─ public-1/
│  └─ index.html
└─ server.js
```
index.htmlファイルに以下の内容を記述する。
```
<!DOCTYPE html>
<html>

<!-- headタグの中にはメタデータ等を記載する -->
<head>
  <meta charset="utf-8">
</head>

<!-- bodyタグの中には実際に表示するものなどを書く -->
<body>
  <h1>H1見出しですよ</h1>
</body>
</html>
```
server.jsファイルを以下の内容に書き換える。
```
// localhostにDenoのHTTPサーバーを展開

 Deno.serve(async (request) => {
     const htmlText = await Deno.readTextFile("./public/index.html");
      return new Response(
          // Responseの第一引数にレスポンスのbodyを設置
         htmlText,
          // Responseの第二引数にヘッダ情報等の付加情報を設置
          {
              // レスポンスにヘッダ情報を付加
  ...
```
ブラウザを再読み込みし、「H1見出しですよ」と表示される。  


##### 1.8.2 CSSファイルを読み込むようにする。  
public-1フォルダの中に、styles.cssを作成します。
```
├─ .vscode/
├─ public-1/
│  ├─ index.html
│  └─ styles.css
└─ server.js
```
上記のような状態にする。  

各ファイルを以下のようにする。  
CSS
```
/* public-1/styles.css */
body {
    background: skyblue;
}
```
HTML  
```
<!-- public-1/index.html -->
  ...
  <!-- headタグの中にはメタデータ等を記載する -->
  <head>
    <meta charset="utf-8">
   <link rel="stylesheet" href="styles.css">
  </head>

  <!-- bodyタグの中には実際に表示するものなどを書く -->
  ...
```
server.js
```
// server.js
  ...
  // localhostにDenoのHTTPサーバーを展開
  Deno.serve(async (request) => {
     // パス名を取得する
     // http://localhost:8000/hoge に接続した場合"/hoge"が取得できる
     const pathname = new URL(request.url).pathname;
     console.log(`pathname: ${pathname}`);
 
     // http://localhost:8000/styles.css へのアクセス時、"./public/styles.css"を返す
     if (pathname === "/styles.css") {
         const cssText = await Deno.readTextFile("./public/styles.css");
         return new Response(
             cssText,
             {
                 headers: {
                     // text/css形式のデータで、文字コードはUTF-8であること
                     "Content-Type": "text/css; charset=utf-8"
                 }
             }
         );
     }
 
      const htmlText = await Deno.readTextFile("./public/index.html");
      return new Response(
          // Responseの第一引数にレスポンスのbodyを設置
  ...
```

ブラウザを再読み込みしたら、背景が青くなっている。  


##### 1.8.3 public-1フォルダ全体を公開してみる

server.jsを以下の内容に書き換えて保存する。  

```
// deno.landに公開されているモジュールをimport
  // denoではURLを直に記載してimportできます
 import { serveDir } from "https://deno.land/std@0.223.0/http/file_server.ts";

  // localhostにDenoのHTTPサーバーを展開
  Deno.serve(async (request) => {
      // パス名を取得する
      // http://localhost:8000/hoge に接続した場合"/hoge"が取得できる
      const pathname = new URL(request.url).pathname;
      console.log(`pathname: ${pathname}`);

     // ./public-1以下のファイルを公開
     return serveDir(
         request,
         {
             /*
             - fsRoot: 公開するフォルダを指定
             - urlRoot: フォルダを展開するURLを指定。今回はlocalhost:8000/に直に展開する
             - enableCors: CORSの設定を付加するか
             */
             fsRoot: "./public-1/",
             urlRoot: "",
             enableCors: true,
         }
     );
  
  });
```
ブラウザを再読み込みして、先程と同じ内容が表示される。  
#### 1.9 ブラウザでJavaScriptを実行する
最後に、ブラウザでJavaScriptを実行してみる。  
public-1/index.htmlファイルを以下の内容に書き換える。  
```
<!-- bodyタグの中には実際に表示するものなどを書く -->
  <body>
    <h1>H1見出しですよ</h1>
 
   <!-- JavaScriptを実行 -->
   <script>
     alert("Hello JavaScript!");
   </script>
 </body>
  
  </html>
```
ブラウザを再読み込みし、アラートが表示されれば、プログラムは正常である。  

環境構築手順は以上である。  

### 2. しりとりの実装
さて、今回の目玉であるしりとりを実装する。  

#### 2.0.1  GET /shiritoriを実装する

server.jsを以下の内容に書き換えて保存する。
```
...
  import { serveDir } from "https://deno.land/std@0.223.0/http/file_server.ts";
  
  // 直前の単語を保持しておく
  let previousWord = "しりとり";
  
  // localhostにDenoのHTTPサーバーを展開
  Deno.serve(async (request) => {
      // パス名を取得する
      // http://localhost:8000/hoge に接続した場合"/hoge"が取得できる
      const pathname = new URL(request.url).pathname;
      console.log(`pathname: ${pathname}`);
  
      // GET /shiritori: 直前の単語を返す
      if (request.method === "GET" && pathname === "/shiritori") {
          return new Response(previousWord);
      }
 
      // ./public-1以下のファイルを公開
      return serveDir(
          request,
  ...	
```
ブラウザで<http://localhost:8000/shiritori>にアクセスすると、「しりとり」と表示される。  

#### 2.0.2. POST /shiritoriを実装する

server.jsを以下の内容に書き換えて保存する。

```
...
// GET /shiritori: 直前の単語を返す
      if (request.method === "GET" && pathname === "/shiritori") {
          return new Response(previousWord);
      }
  
      // POST /shiritori: 次の単語を入力する
      if (request.method === "POST" && pathname === "/shiritori") {
          // リクエストのペイロードを取得
          const requestJson = await request.json();
          // JSONの中からnextWordを取得
          const nextWord = requestJson["nextWord"];

          // previousWordの末尾とnextWordの先頭が同一か確認
          if (previousWord.slice(-1) === nextWord.slice(0, 1)) {
              // 同一であれば、previousWordを更新
              previousWord = nextWord;
          }
  
         // 現在の単語を返す
          return new Response(previousWord);
      }
 
      // ./public以下のファイルを公開
      return serveDir(
          request,
...
```

#### 2.1. しりとりの実装: Webの処理を実装する

##### 2.1.1. GET /shiritoriの結果を表示する
GET /shiritoriにアクセスして、直前の単語を取得する。  

以下のようにして実装する。  

public-1/index.htmlファイルを以下の内容に書き換える。  
```
...
  <!-- bodyタグの中には実際に表示するものなどを書く -->
  <body>

    <h1>しりとり</h1>
    <!-- 現在の単語を表示する場所 -->
    <p id="previousWord"></p>
  
    <!-- JavaScriptを実行 -->
    <script>

      window.onload = async (event) => {
        // GET /shiritoriを実行
        const response = await fetch("/shiritori", { method: "GET" });
        // responseの中からレスポンスのテキストデータを取得
        const previousWord = await response.text();
        // id: previousWordのタグを取得
        const paragraph = document.querySelector("#previousWord");
        // 取得したタグの中身を書き換える
        paragraph.innerHTML = `前の単語: ${previousWord}`;
      }
    </script>
  </body>
...
```
ブラウザをhttp://localhost:8000で再読み込みして、「しりとり」と表示される。

#### 2.1.2. POST /shiritoriに次の単語を送信する
POST /shiritoriにアクセスして、次の単語を入力する。  
ここでは、ブラウザの起動時に勝手に「りんご」と送信されるようにしてみる。  


public-1/index.htmlファイルを以下の内容に書き換える。  
```
 <!-- JavaScriptを実行 -->
    <script>
      window.onload = async (event) => {
       // 試しでPOST /shiritoriを実行してみる
       // りんごと入力……
       await fetch(
         "/shiritori",
         {
           method: "POST",
           headers: { "Content-Type": "application/json" },
           body: JSON.stringify({ nextWord: "りんご" })
         }
       );
 
        // GET /shiritoriを実行
        const response = await fetch("/shiritori", { method: "GET" });
        // responseの中からレスポンスのテキストデータを取得
```
ブラウザを再読み込みして、「りんご」と表示される。

#### 2.1.3. POST /shiritoriに任意の単語を送信してみる
public-1/index.htmlファイルを以下の内容に書き換える。  
送信ボタンが押下された時にinputタグの中身を取得して、POST /shiritoriに送信する。
```
<h1>しりとり</h1>
    <!-- 現在の単語を表示する場所 -->
    <p id="previousWord"></p>
    <!-- 次の文字を入力するフォーム -->
    <input id="nextWordInput" type="text" />
    <button id="nextWordSendButton">送信</button>
  
    <!-- JavaScriptを実行 -->
    <script>
      window.onload = async (event) => {
        // GET /shiritoriを実行
        const response = await fetch("/shiritori", { method: "GET" });
        // responseの中からレスポンスのテキストデータを取得
        const previousWord = await response.text();
        // id: previousWordのタグを取得
        const paragraph = document.querySelector("#previousWord");
        // 取得したタグの中身を書き換える
        paragraph.innerHTML = `前の単語: ${previousWord}`;
      }
  
      // 送信ボタンの押下時に実行
      document.querySelector("#nextWordSendButton").onclick = async(event) => {
        // inputタグを取得
        const nextWordInput = document.querySelector("#nextWordInput");
        // inputの中身を取得
        const nextWordInputText = nextWordInput.value;
        // POST /shiritoriを実行
        // 次の単語をresponseに格納
       const response = await fetch(
          "/shiritori",
          {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ nextWord: nextWordInputText })
          }
        );
  
        const previousWord = await response.text();
  
        // id: previousWordのタグを取得
        const paragraph = document.querySelector("#previousWord");
        // 取得したタグの中身を書き換える
        paragraph.innerHTML = `前の単語: ${previousWord}`;
        // inputタグの中身を消去する
        nextWordInput.value = "";
      }
    </script>
  </body>
```
ブラウザを読み込み直して、入力フォームが表示される。

#### 2.2. しりとりの実装: エラーを実装する
「りんご」の次に「らっぱ」などの続かない単語が入力された時に、エラーを表示できるようにする。  
ここでは、application/json形式のデータを返すようにする。  

server.jsファイルを以下の内容に書き換える。
```
...
// POST /shiritori: 次の単語を入力する
      if (request.method === "POST" && pathname === "/shiritori") {
          // リクエストのペイロードを取得
          const requestJson = await request.json();
          // JSONの中からnextWordを取得
          const nextWord = requestJson["nextWord"];
          // previousWordの末尾とnextWordの先頭が同一か確認
          if (previousWord.slice(-1) === nextWord.slice(0, 1)) {
              // 同一であれば、previousWordを更新
              previousWord = nextWord;
          }
          // 同一でない単語の入力時に、エラーを返す
          else {
              return new Response(
                  JSON.stringify({
                      "errorMessage": "前の単語に続いていません",
                      "errorCode": "10001"
                  }),
                  {
                      status: 400,
                      headers: { "Content-Type": "application/json; charset=utf-8" },
                  }
              );
          }
...
```

public-1/index.htmlファイルを以下の内容に書き換える。
```
...
// 送信ボタンの押下時に実行
      document.querySelector("#nextWordSendButton").onclick = async(event) => {
        // inputタグを取得
        const nextWordInput = document.querySelector("#nextWordInput");
        // inputの中身を取得
        const nextWordInputText = nextWordInput.value;
        // POST /shiritoriを実行
        // 次の単語をresponseに格納
        const response = await fetch(
          "/shiritori",
          {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ nextWord: nextWordInputText })
          }
        );
 
       // status: 200以外が返ってきた場合にエラーを表示
       if (response.status !== 200) {
         const errorJson = await response.text();
         const errorObj = JSON.parse(errorJson);
         alert(errorObj["errorMessage"]);
       }
  
        const previousWord = await response.text();
...
```
ブラウザを再読み込みして、不正な単語を入力してみると、アラートが表示される。

以上の内容を実装すると  

直前の単語が表示できる。  
次の単語を入力できる。  
直前の単語の末尾と入力した単語の先頭が同一であれば、単語を更新。同一でなければ、エラーを表示する。  

ができるようになる。

あとは、自分で処理を考え実装する。


### 3. 実装機能
実装した内容は、以下の通りである。  

・直前の単語を、表示できるようにする。  
・任意の単語を、入力できるようにする。  
・直前の単語の末尾と、入力した単語の先頭を比較して、同じ場合だけ単語を更新する。違う場合は、エラーを表示する。  
・末尾が「ん」で終わる単語が入力されたら、ゲームを終了する。  
・過去に使用した単語が入力されたら、ゲームを終了する。  
・ゲーム中や終了後に、最初からやり直せるリセット機能をつける。  
・最初の単語がランダムに決まるようにする。  
・絵文字等、しりとりとして不適切な単語は入力できないようにする。  
・ひらがなとカタカナを入力できる。  
・Enterキーを押すことで送信できるようにする。  
・Shiftキーを押すことでリセットできるようにする。  

以上
### 4. ルールの概要

「ん」で終わる単語やリセットや過去に使用した単語を入力したら、リセットボタンを押す。  
カタカナを実装した理由は、スマホなどのカタカナ用語をひらがなで入力すると、おかしいと思ったからである。  
最後に、漢字は入力しないこと。  

### 5. 改良点

改良点として、複数のユーザーが対戦できるようにすることを実装したかったところと  
実在しない単語は入力できないようにすることが実装できたら面白いと思った。
