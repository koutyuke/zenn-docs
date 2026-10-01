---
title: "ChatGPTでアーカイブしていないチャットをすべて削除する"
emoji: "🗑️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ChatGPT"]
published: true
---

こんにちは、こうちゅけです。
ChatGPT使ってますか？ 私は課金ユーザーなのでそれなりに使っています。
それなりに使っているとチャットのログがたまり続けて「一気に削除したい!」と思うこともしばしばあると思います。

![ChaGPTの画面](https://storage.googleapis.com/zenn-user-upload/e3eaaef925fa-20250305.png)
_左にどんどんチャットが溜まっていく_

ChatGPTでは `[設定] → [一般] → [すべてのチャットを削除する]` でチャットをすべて削除することができます。しかし、これを行うとアーカイブしているチャットもすべて消されてしまいます。

そこで今回はちょっとズルをしてアーカイブしていないチャット(サイトを開いたときに左の欄に出てきている奴ら)をまとめて消す事ができたのでご紹介します。

# アクセストークンを取得する

まず最初にアクセストークンを取得していきます。
そんなことできるん？と思うと思いますが、ズルをします。

ブラウザで`ChatGPT`のサイトを開いて、開発者モードを開きます。
エンジニアであれば、この先で行うことはわかると思いますが通信からトークンを取得します。(ズル)

セキュリティ的にどうなのかと言われると何も返せないのですが致し方なし...

開発者モードを開いたら `ネットワーク` タブを開きサイトを更新します。

![開発者モード](https://storage.googleapis.com/zenn-user-upload/26058401b734-20250305.png)

そこからどの通信でもいいのですが `https://chatgpt.com/backend-api` に対してfetchしているものを開き、`ヘッダー` タブ、`リクエストヘッダー`、`Authorization`項目にある `Bearer ・・・` をまとめてコピーします。

これでトークンが取得できました。

# チャットを削除する

最後にチャットを削除していきます。

開発者モードの画面で `コンソール` を開き以下のコードを実行します。

![コンドール画面](https://storage.googleapis.com/zenn-user-upload/50b3b9eb3879-20250305.png)

```js
async function deleteChat() {
  const token = "Bearer ..."; // 先ほど取得したトークンに置き換える
  try {
    const response = await fetch("https://chatgpt.com/backend-api/conversations?offset=0", {
      method: "GET",
      headers: {
        authorization: token,
      },
      credentials: "include",
    });

    if (!response.ok) {
      throw new Error(`HTTP error! Status: ${response.status}`);
    }

    const data = await response.json();

    for (const item of data.items) {
      await fetch(`https://chatgpt.com/backend-api/conversation/${item.id}`, {
        method: "PATCH",
        headers: {
          authorization: token,
          "content-type": "application/json",
        },
        credentials: "include",
        body: JSON.stringify({ is_visible: false }),
      });

      // 0.1秒 (100ミリ秒) 待機
      await new Promise(resolve => setTimeout(resolve, 100));
    }
  } catch (error) {
    console.error("Error fetching conversations:", error);
  }
}

// 関数を実行
deleteChat();
```

これで特にエラーが起きなければチャットが消されているはずです。

# 解説

軽くコードを解説していきます。

まずチャット一覧を取得していきます。

```js
await fetch("https://chatgpt.com/backend-api/conversations?offset=0", {
  method: "GET",
  headers: {
    authorization: token,
  },
  credentials: "include",
});
```

そしてレスポンスがチャットの情報があるオブジェクトの配列になっているのでそれをfor文で回しています。

for文の中では削除(非表示)にするコードを実行しています。

```js
await fetch(`https://chatgpt.com/backend-api/conversation/${item.id}`, {
  method: "PATCH",
  headers: {
    authorization: token,
    "content-type": "application/json",
  },
  credentials: "include",
  body: JSON.stringify({ is_visible: false }),
});
```

これで表示されているものは消されるってわけです。

# 最後に

ChatGPTでアーカイブしていないチャットを削除(非表示)にする方法をご紹介しました。
もし、他にいい方法がある場合は教えて下さい!
