layout: true
class: center, middle

---
background-image: url(images/lambda.png)
# AWS Lambda@Edge<br/>について
### CM re:Growth 2016 OSAKA<br/>2016/12/12 中山 幸治

---
# 中山 幸治(knakayama)&nbsp;![knakayama](https://avatars2.githubusercontent.com/u/1545919?v=3&s=400)
### [http://dev.classmethod.jp/author/nakayama-koji/](http://dev.classmethod.jp/author/nakayama-koji/)<br/>[https://knakayama.github.io](https://knakayama.github.io)
### ![PSA](images/PSA.png)

---
# クラスメソッド![classmethod](https://pbs.twimg.com/profile_images/344513261567475947/516b99bf0b40c0354dfad345ef14051a_400x400.png)<br/>AWSコンサルティング部<br/>所属

---
layout: true
class: middle

---
### 1. 今までの課題
### 2. AWS Lambda@Edgeとは
### 3. ユースケース
### 4. 利用する上での注意点
### 5. 参考リンク

---
layout: true
class: center, middle

---
# 1. 今までの課題

---
# CloudFrontなどのCDNを利用している場合オリジンまでアクセスさせてしまうとネットワークレイテンシが高くなる

---
layout: true
class: middle

---
### - リクエストの内容によって動的な処理をしたい場合はある
### - 例えばユーザエージェントに応じて返答するオブジェクトを変更するなど
### - この処理はオリジン側でないと実装が難しい
### - だけどそれだと遅い
### - あとサーバの管理したくない

---
layout: true
class: center, middle

---
# 2. AWS Lambda@Edge<br/>とは

---
# CloudFrontの各種イベントに基づきLambda関数をエッジロケーションで実行させ、リクエスト/レスポンスオブジェクトを動的に処理できる機能

---
layout: true
class: middle

---
### - エッジロケーションで実行されるので低レイテンシを実現
### - イベント駆動でLambda関数を実行できるためサーバーレス
### - イベントには今のところ以下4つがある<br/>&nbsp;&nbsp;&nbsp;&nbsp;- Viewer Request<br/>&nbsp;&nbsp;&nbsp;&nbsp;- Viewer Response<br/>&nbsp;&nbsp;&nbsp;&nbsp;- Origin Request</br>&nbsp;&nbsp;&nbsp;&nbsp;- Origin Response
### - リクエスト/レスポンスイベントによって `event` オブジェクトに渡されたデータをLambda関数で処理可能

---
### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;リクエストオブジェクト

```json
{
    "Records":[
          {
              "cf": {
                  "configuration": {
                      "distributionId": "EXAMPLE"
                  },
                  "request": {
                      "uri": "/me.pic",
                      "method": "GET",
                      "httpVersion": "2.0",
                      "clientIp": "2001:cdba::3257:9652",
                      "headers": {
                          "User-Agent": ["Test Agent"],
                          "Host" : ["d2fadu0nynjpfn.cloudfront.net"]
<snip>
```

---
### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;レスポンスオブジェクト

```json
{
    "Records":[
          {
              "cf": {
                  "configuration": {
                      "distributionId": "EDFDVBD6EXAMPLE "
                  },
                  "response": {
                      "status": "200",
                      "statusDescription": "HTTP OK",
                      "httpVersion": "2.0",
                      "headers": {
                          "User-agent": [ "mozilla", "safari" ],
                          "Vary": [ "*" ]
<snip>
```

---
### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ヘッダの書き換え(ブループリントから引用)

```javascript
'use strict';
exports.handler = (event, context, callback) => {
    const response = event.Records[0].cf.response;
    const headers = response.headers;
    const customHeaderName = 'X-Amz-Meta-Last-Modified';
    const headerNameToBeChanged = 'Last-Modified';

    if (headers[customHeaderName]) {
        headers[headerNameToBeChanged] = headers[customHeaderName];
    }
    callback(null, response);
};
```

---
layout: true
class: center, middle

---
# 3. ユースケース

---
layout: true
class: middle

---
### - A/Bテスト<br/>&nbsp;&nbsp; - 特定のクッキーが存在した場合に返却するオブジェクトを動的に変更させる
### - ユーザエージェント毎に最適なオブジェクトの返却<br/>&nbsp;&nbsp; - 例えばスマホならXXの解像度の画像ファイルを返すなど
### - アクセスコントロール<br/>&nbsp;&nbsp; - ヘッダの内容に応じてオブジェクトへのアクセス権を制御
### - ヘッダ/URLなどの動的な変更

---
layout: true
class: center, middle

---
# 4. 利用する上での注意点

---
layout: true
class: middle

---
### - 現時点(2016/12/12)ではプレビュー扱い
### - 機能に制限あり<br/>&nbsp;&nbsp;&nbsp;&nbsp;- Node.js 4.3のみサポート<br/>&nbsp;&nbsp;&nbsp;&nbsp;- HTTP(S)リクエスト/レスポンスヘッダの読み書き/URLの<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;書き換えのみ可(プレビュー版)<br/>&nbsp;&nbsp;&nbsp;&nbsp;- ローカルファイルシステムへの書き込み不可<br/>&nbsp;&nbsp;&nbsp;&nbsp;- there are no built-in libraries available<br/>&nbsp;&nbsp;&nbsp;&nbsp;- 最大メモリ128MB<br/>&nbsp;&nbsp;&nbsp;&nbsp;- 最大実行時間50ms<br/>&nbsp;&nbsp;&nbsp;&nbsp;- 最大デプロイパッケージサイズ1MB<br/>&nbsp;&nbsp;&nbsp;&nbsp;- 最大グローバルTPS 100

---
layout: true
class: center, middle

---
# 5. 参考リンク

---
layout: true
class: middle

---
### - AWS公式ドキュメント
#### - http://docs.aws.amazon.com/lambda/latest/dg/lambda-edge.html<br/>- http://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/lambda-at-the-edge.html
### - AWS Blog
#### - https://aws.amazon.com/blogs/aws/coming-soon-lambda-at-the-edge/
### - プレビュー参加申請ページ
#### - https://pages.awscloud.com/lambda-at-edge-preview.html
### - Developers.IOのLambda@Edge関連エントリ
#### - http://dev.classmethod.jp/tag/aws-lambdaedge/

---
layout: true
class: center, middle

---
# おわり
