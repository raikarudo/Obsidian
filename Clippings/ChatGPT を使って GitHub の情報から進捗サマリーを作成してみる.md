---
title: "ChatGPT を使って GitHub の情報から進捗サマリーを作成してみる"
source: "https://qiita.com/shyamagu/items/5709021e7b347ebd4f8a"
author:
  - "[[Qiita]]"
published: 2023-03-20
created: 2025-05-13
description: "はじめにChatGPT すごいですよね(略）前回の記事では、ChatGPT をつかった業務システムっぽい構成を検証してみましたが、今回はもうちょっと現実的に使えそうな方向ということで、要約機能を…"
tags:
  - "clippings"
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=)

## Qiitaにログインして、便利な機能を使ってみませんか？

[ログイン](https://qiita.com/login?callback_action=login_or_signup&redirect_to=%2Fshyamagu%2Fitems%2F5709021e7b347ebd4f8a&realm=qiita) [新規登録](https://qiita.com/signup?callback_action=login_or_signup&redirect_to=%2Fshyamagu%2Fitems%2F5709021e7b347ebd4f8a&realm=qiita)

この記事は最終更新日から1年以上が経過しています。

## はじめに

ChatGPT すごいですよね(略）  
[前回の記事](https://qiita.com/shyamagu/items/19c6e3c7dda54320aa36) では、ChatGPT をつかった業務システムっぽい構成を検証してみましたが、今回はもうちょっと現実的に使えそうな方向ということで、要約機能を試したいと思います。

週次レビューでどんな作業をしてきたか、スクラムの朝会で昨日何したか、あたりを GitHub の commit log の情報を ChatGPT に与えて要約してみたいと思います。

今回は諸事情で Azure OpenAI Service を使いますが、基本的な使い方は OpenAI と一緒です。適宜読みかえてもらえたらなと思います。  
なお、使ったモデルは GPT-3.5-TURBO (0301) です。(早く 4 でやってみたい。。。)

## GitHub の commit log の加工

パラメータは色々ありますが、今回は以下の形式で、sinceとuntilを指定したいと思います。

```text
https://api.github.com/repos/{owner}/{repo}/commits?since=2020-01-01T00:00:00+09:00&until=2020-12-31T23:59:59+09:00
```

これだと {owner}/{repo} の 2020年1月1日0時0分0秒~2020年12月31日23時59分59秒（日本時間+9）  
の commit 履歴を全部抜いてくることができます。

ただこの commit 履歴には様々な情報が含まれるため、これをそのまま ChatGPT に渡すと token の盛大な無駄遣いになること＆余計な情報が要約に悪影響しそうなため、API を呼び出した結果から必要な情報(author.name と commit.message、あと author.date) だけ抜きます。  
**前回の記事で作ったアプリを元にしたため、サンプルコードは、Vue3.2(CDN) + Expressjs での実装抜粋です**

あと今回は public なリポジトリを対象にしました。 private の場合は認証を通してあげれば同じことができます。が、ここでは割愛します。

```js
fetch("https://api.github.com/repos/{owner}/{repo}/commits?since=2020-01-01T00:00:00+09:00&until=2020-12-31T23:59:59+09:00")
    .then(response => response.json())
    .then(data => {
        const commits = data.map((item)=>({
            "name": item.commit.author.name,
            "message": item.commit.message,
            "date":item.commit.author.date
        }))
        this.message = JSON.stringify(commits)
    })
```

ここで取得した this.message を Azure OpenAI Servcie にポストする形にしました。

## ChatGPT の AI モデルについて

Azure OpenAI Service の場合、以下のパラメータを指定します。

- {Azure OpenAI Endpoint}
- {Deployment Model}
- {Azure OpenAI API Key}  
	それぞれのパラメータについては、 [公式のドキュメント](https://learn.microsoft.com/ja-jp/azure/cognitive-services/openai/chatgpt-quickstart?tabs=command-line&pivots=rest-api) を参考にしました。  
	また、もしOpenAI の方を使うのであれば、前回記事のように、 [公式サイト](https://platform.openai.com/docs/guides/chat/chat-vs-completions) のSDKベースで実装するといいですね。

前回は ChatGPT に日本語で指示を与えていましたが、どうしても回答を固定フォーマットにすることが安定しなかったので、今回は英語＋temperatureを0.5 にしてやってみました。この辺も細かい調整ノウハウありますね・・

なお、やっている内容は、

```json
[
    {"name":NAME1,"message":MESSAGE1,"date":DATE1},
    {"name":NAME2,"message":MESSAGE2,"date":DATE2},
    {"name":NAME1,"message":MESSAGE3,"date":DATE3}
]
```

のスタイルで受信したコミット履歴の情報を、NAMEごとにMESSAGEs を要約してくれ、という指示と、回答は、以下のフォーマットであることを指定する指示をいれました。

```json
[
    {
        "name":NAME1,
        "summary": **SUMMARY OF NAME1's messages in Japanese**
    },
    {
        "name":NAME2,
        "summary": **SUMMARY OF NAME2's messages in Japanese**
    },
]
```

メソッドの全体像は以下の通りです。

```js
async function getAOAIAnswer(message) {

    const response = await axios.post({Azure OpenAI Endpoint }+"/openai/deployments/{Deployment Model}/completions?api-version=2022-12-01", {
        "prompt": \`
        <|im_start|>system\n
        You receive a JSON list like [{"name":NAME1,"message":MESSAGE1,"date":DATE1},{"name":NAME2,"message":MESSAGE2,"date":DATE2},{"name":NAME1,"message":MESSAGE3,"date":DATE3}].
        Please summarize the list by each person and output a JSON list only.
        Your output JSON format MUST BE below. please return **ONLY A JSON LIST**.
[
    {
        "name":NAME1,
        "summary": **SUMMARY OF NAME1's messages in Japanese**
    },
    {
        "name":NAME2,
        "summary": **SUMMARY OF NAME2's messages in Japanese**
    },
]
        <|im_end|>
        <|im_start|>user\n
        ${message}
        <|im_end|>
        <|im_start|>assistant\`,
        "max_tokens": 800,
        "temperature": 0.5,
        "frequency_penalty": 0,
        "presence_penalty": 0,
        "top_p": 0.95,
        "stop": ["<|im_end|>"]
        }, {
        headers: {
        'Content-Type': 'application/json',
        'api-key': {Azure OpenAI API Key}
        }
      });

    return response.data.choices[0].text
```

## サンプルアプリでの実行結果

それでは上記の内容をもとに、public なリポジトリということで、今回は [dotnet/aspnetcore](https://github.com/dotnet/aspnetcore/) のコミット履歴を使わせてもらいたいと思います。

選んだ基準は、コミットが頻繁に行われていること、コミットログがキレイに入っていることでした。色々他のプロジェクトのも見てみたいところです。

実際に試したのは、日本時間の2023年3月1日9時～2023年3月2日9時までの24時間のデータです。  
こんなのです-> https://api.github.com/repos/dotnet/aspnetcore/commits?since=2023-03-01T18:00:00Z&until=2023-03-02T18:00:00Z

取得できたコミットログを必要なデータのみに絞って、ChatGPT に要約してもらった結果が以下です。きちんと全部確認はしてませんが、それっぽく要約してくれていますね！

```json
[
    {
        "name": "Brennan",
        "summary": "client result fail でクライアントが呼び出し引数を解析できない場合は失敗する(#46910)"
    },
    {
        "name": "Nick Stanton",
        "summary": "QuickGridを製品化する(#46573)"
    },
    {
        "name": "dotnet-maestro[bot]",
        "summary": "dotnet / efcoreビルド20230301.1から依存関係を更新する(#46966) [main] dotnet / efcoreから依存関係を更新する(#46930)"
    },
    {
        "name": "William Godbe",
        "summary": "sourceIndexPackageVersionを更新する(#46946)"
    },
    {
        "name": "Thays Grazia",
        "summary": "_hotReloadAgent変数を初期化する前に使用しようとしないようにする(#46916)"
    },
    {
        "name": "Bruno Oliveira",
        "summary": "ProblemDetailsJsonOptionsSetupでJsonSerializerOptionsコピーを削除する(#46716)"
    },
    {
        "name": "Mitch Denny",
        "summary": "ルートパラメーターのTryParseサポートを追加する(#46921)"
    }
]
```

今回は英語で指示をしたからか、JSONフォーマットの返り値はかなり確実に（試している限りは安定して）返ってきました。JSONで返ってくるなら Web 上に綺麗に表示できますねやったー。

[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/977156/64677b0f-d8d9-61cb-2d52-4d0eabc82116.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F977156%2F64677b0f-d8d9-61cb-2d52-4d0eabc82116.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=98141a6e156304decdaa828440a8e1b1)

ちょっとおまけ的に、 [shyamagu/choi](https://github.com/shyamagu/choi) のコミットログも要約してみます。  
こんなURLをもとにします。 -> https://api.github.com/repos/shyamagu/choi/commits?since=2022-06-01T18:00:00Z&until=2022-07-01T18:00:00Z  
2022年6月1日9時～7月1日9時までの1か月間のデータです。（そもそもコミットが少ないので、期間を長くして対応）

同じように以下の結果を得ることができました。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/977156/d01a951e-e6e2-6e04-1e5f-ae11dd7aa3b8.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F977156%2Fd01a951e-e6e2-6e04-1e5f-ae11dd7aa3b8.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=14b5443ccb1915a07911c1def94fa118)

最近なのか？というのはありますが、何をやったかは何となくわかりますね。いいですね。

## まとめと考察

今回は、GitHub のコミットログを ChatGPT に与えて、要約してもらうことに挑戦してみました。結構いい感じにまとめが出てるかな？と思いましたが、課題としては

- コミットログが適当だとどうしようもない ("hoge"とか"test"とか入れられてると…）
- コミットログが少なければ、別に要約なんかしなくても GitHub 上で確認すればよい
- きちんと要約ができるとして、コスパの観点はどうなんだろう？（大したコストではないですが…）

っていうところでしょうか。逆に、

- コミットが多人数で、頻繁に実施されている
- コミットログがきちんと書かれている

のであれば、パッと確認するのにはとても良さそうかなと思いました。

Private リポジトリ―に対しても GitHub Apps とかつかって認証とれば取得できますので、レビュー時に表示して、あとは個々人に補足してもらう使い方をすると良いかもですよね。もし ChatGPT がきちんと要約できていなくても、その場で訂正できるのであれば使い方として間違いないかなとも。（あると嬉しい系）

あと今回は英語で実施したことで JSON フォーマットが安定して返ってきたもの面白いポイントでした。GPT-4 とかになれば日本語でも問題ないようになるかもしれませんが、それでも英語の方が精度は良いらしいので、やっぱり英語かぁっていう。  
それでもたまに回答が途中で途切れる事象は発生しました。今回のつくりだとJSON にパースできなかったら再取得とかするといいかもです。

あとそうだ。summary の内容がやっぱり実行の都度かなり揺れます。ここについては、どんな summary なのかもう少し補足してあげれば良さそうです。

というわけで、もし GitHub をつかっていて、かつレビュなどのタイミングで誰が何をしたかをコミットベースで会話している（人間系でまとめている）のであれば、役に立ちそうな感じがしましたということで、今回の検証はここで終わりです。読んでいただき、ありがとうございました。

[0](https://qiita.com/shyamagu/items/#comments)

新規登録して、もっと便利にQiitaを使ってみよう

1. あなたにマッチした記事をお届けします
2. 便利な情報をあとで効率的に読み返せます
3. ダークテーマを利用できます
[ログインすると使える機能について](https://help.qiita.com/ja/articles/qiita-login-user)

[新規登録](https://qiita.com/signup?callback_action=login_or_signup&redirect_to=%2Fshyamagu%2Fitems%2F5709021e7b347ebd4f8a&realm=qiita) [ログイン](https://qiita.com/login?callback_action=login_or_signup&redirect_to=%2Fshyamagu%2Fitems%2F5709021e7b347ebd4f8a&realm=qiita)

[1](https://qiita.com/shyamagu/items/5709021e7b347ebd4f8a/likers)

1