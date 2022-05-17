いよいよ最後です。

実際にWebアプリを外部に公開できるよう仕上げていきます。

### **🙉Githubに関するメモ**

**どんなプロジェクトにおいてもGithubにアップロードする場合は、秘密鍵を含む`hardhat.config.js`をリポジトリにアップロードしないでください。**

**資金を奪われてしまいます！**

これから行うフロントエンド側のデプロイには秘密鍵は含まれていませんが、後々のためにここでは`dotenv`というパッケージを使用して秘密鍵をより安全に扱う方法を挙げておきます。

```bash
npm install --save dotenv
```

`hardhat.config.js`を変更します。

```solidity
require('@nomiclabs/hardhat-waffle');
require('dotenv').config();

module.exports = {
  solidity: '0.8.4',
  networks: {
    mumbai: {
      url: process.env.STAGING_ALCHEMY_KEY,
      accounts: [process.env.PRIVATE_KEY],
    },
    mainnet: {
      chainId: 1,
      url: process.env.PROD_ALCHEMY_KEY,
      accounts: [process.env.PRIVATE_KEY],
    },
  },
};
```

.env ファイルの中身を次のように作成します。

```
STAGING_ALCHEMY_KEY=YOURS_1
PRIVATE_KEY=YOURS_2
```
YOURS_1,2のところはご自分のものを使用してください(" "で囲います)。

また、言うまでもないことですが`.env`をコミットしてはいけません。`.gitignore`ファイルに`.env`を入力します.

以前に`.gitignore`に加えた変更を覚えていますか？

これで、 `hardhat.config.js`行を削除して元に戻すことができます。

これは、このファイルには今、実際のキー情報ではなく、キーを表す変数のみが含まれているためです。

`.gitignore`から削除する場合は、そのファイル内に本当に秘密鍵が入力されていないかは入念に確認してください。

削除した場合、そのファイルは**今後追跡対象**になります。

ということは**コミットされる**ということです。

### 🚀デプロイする

Reactアプリのデプロイはとても簡単です。 今回Vercelを使用しますが**無料**です。

Vercel に関する説明は、[こちら](https://zenn.dev/lollipop_onl/articles/eoz-vercel-pricing-2020)などをご覧ください。

手順について、詳しくは別のプロジェクト Polygon-Generative-NFT Section 4 - Lesson 2 を参照ください。

<br/>

これまでにアプリを完成させてきました。

いよいよこのデプロイが最後のステップです。



- 最新のフロントエンドコードをGithubにプッシュします。 (`.env`など公開したくないファイルがある場合そのファイルはコミットしないでください。)
- VercelをGithubのリポジトリに接続します。VercelのDashboardページの右上に見える`New Project`ボタンから登録していきます。
- 設定を入力（下の注を参照）したらデプロイします。
- 完成です。

注：Vercelでは、環境変数 `CI=false`を追加する必要があります。 これにより、`warning`が原因でビルドが失敗しないようになります。

![](/public/images/14-Polygon-ENS/section-4/4_3_1.png)


設定を入力したら`deploy`ボタンを押してください。

デプロイにはしばらく時間がかかります。

<br/>

・・・

お待たせしました。

皆さんの作成したアプリがついにデプロイできました！！！

あなたのアプリを世界中の人が使うことができます！



これで、ドメインサービスについてはもう習熟されています🎉


![](/public/images/14-Polygon-ENS/section-4/4_3_2.png)


### 🎉 おつかれさまでした！

あなたのオリジナルのドメインネームサービスWebアプリケーションが完成しました。

あなたが習得したスキルは、分散型 Web アプリがより一般的になる社会の中で、世界を変える重要なスキルです。

これからも Web3 への旅をあなたが続けてくれることを願っています 🚀

### 🎫 NFT を取得しよう！

NFT を取得する条件は、以下のようになります。

1. MVP の機能がすべて実装されている（実装 OK）

2. Web アプリケーションで MVP の機能が問題なく実行される（テスト OK）

3. このページの最後にリンクされている Project Completion Form に記入する

4. Discord の `🔥｜post-your-project` チャンネルに、あなたの Web サイトをシェアしてください 😉🎉 Discord に投稿する際に、追加実装した機能とその概要も教えていただけると幸いです！

プロジェクトを完成させていただいた方には、NFT をお送りします。


---
お疲れ様でした！！！
ぜひUNCHAINの他のプロジェクトの学習もしてみてください。