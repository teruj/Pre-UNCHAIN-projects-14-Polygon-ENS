スマートコントラクトができてきました。

ただし、現在はマッピングを提供しているだけです。

ウォレットや OpenSea で実際に表示することはできません。

これから行うことは、**ドメインを OpenSea で表示可能な NFT に変換**し、ドメインの長さに応じてさまざまな金額で販売することです。

ちなみに、ENS ドメインでは一度に特定のドメインを保持できるウォレットは 1 つだけです。

### 💰 支払いを実装しよう

私たちは実際には `.eth`のような TLD（トップレベルドメイン）をコントラクトに入れていません。

ミントしたいドメインを設定しましょう！ 私は`.ninja`を使います 🥷

**ご自身のものを設定してみてください。**。 「.takeshi」など何でも結構です：

`Domains.sol` を変更します。

```javascript
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.4;

// インポートを忘れずに。
import { StringUtils } from "./libraries/StringUtils.sol";

import "hardhat/console.sol";

contract Domains {
  // トップレベルドメイン(TLD)です。
  string public tld;

  mapping(string => address) public domains;
  mapping(string => string) public records;

  // constructorに"payable"を加えます。
  constructor(string memory _tld) payable {
    tld = _tld;
    console.log("%s name service deployed", _tld);
  }

  // domainの長さにより価格が変わります。
  function price(string calldata name) public pure returns(uint) {
    uint len = StringUtils.strlen(name);
    require(len > 0);
    if (len == 3) { // 3文字のドメインの場合 (通常,ドメインは3文字以上とされます。あとのセクションで触れます。)
      return 5 * 10**17; // 5 MATIC = 5 000 000 000 000 000 000 (18ケタ).あとでfaucetから少量もらう関係 0.5MATIC。
    } else if (len == 4) { //4文字のドメインの場合
      return 3 * 10**17; // 0.3MATIC
    } else {
      return 1 * 10**17;
    }
  }

  function register(string calldata name) public payable{
    require(domains[name] == address(0));
    uint _price = price(name);

    // トランザクションを処理できる分だけのMATICがあるか確認
    require(msg.value >= _price, "Not enough Matic paid");

    domains[name] = msg.sender;
    console.log("%s has registered a domain!", msg.sender);
  }
  // 他のfunction は変更せず。
}
```

_注：_ `getAddress`、` setRecord` _、および_ `getRecord` _関数は引き続き必要です。これらは変更されていないため、これを短くするために削除しました。_

ここで新しい言葉に気付くでしょう。

`register`に`payable`を追加しました。

```javasctipt
uint _price = price(name);
require(msg.value >= _price, "Not enough Matic paid");
```

ここでは、送信された`msg`の`value`が一定量を超えているかどうかを確認します。 `value`は送信された Matic の量であり、`msg`はトランザクション本体です。

これは実はすごいことです。 数行のコードで、アプリに支払い機能を追加できます。 API キーは必要ありません。 大手プロバイダーとやりとりする必要もありません。

これがブロックチェーンのすばらしいところです。

また、トランザクションに十分な Matic がない場合、トランザクションは元に戻され、何も変更されません。

`price`関数を詳しく見ると、これは`pure`関数であることがわかります。

つまり、コントラクトの状態を読み取ったり変更したりすることはありません。

価格はJavaScript などを使用してフロントエンドでも実行できますが、その必要もないでしょう。ここでは、チェーン上で最終価格を計算します。

**ドメインの長さに基づいて価格を返すように設定しました。 ドメインが短いほどコストが高くなります**

.com などでもシンプルなドメインは価値が出ますよね。

MATIC トークンには小数点以下 18 桁があるため、価格の最後に `* 10**18`を付ける必要があります。

```javascript
function price(string calldata name) public pure returns(uint) {
  uint len = StringUtils.strlen(name);
  require(len > 0);
  if (len == 3) {
    return 5 * 10**17; // 5 MATIC = 5 000 000 000 000 000 000 (18ケタ).
    //ここではあとでfaucetから少量のMATICをもらう関係で0.5MATICとしておきます。
  } else if (len == 4) {
    return 3 * 10**17; // ドメイン文字数が増えると少し安くなる。0.3MATIC
  } else {
    return 1 * 10**17; //  0.1MATIC
  }
}
```

_注：**Mumbai などテストネットでは価格を下げてミントミントしましょう。** 1 Matic のような設定をするとテストネットの資金がすぐになくなります。 ローカルで実行している場合は何でも課金できますが、実際のテストネットワークを使用している場合は注意が必要です。_

他に、次の 3 つを追加しました。

- `import{StringUtils}` バッケージをインポートしています。 これについては下で説明しています。

- 文字列 `tld` これは、ドメインの末尾を記録します（例：`.ninja`）。

- `string memory _tld` constructor は 1 回だけ実行されます。これは public の`tld`変数を設定する方法です。

`contracts`フォルダーに`libraries`という新しいフォルダーを作成し、`StringUtils.sol`というファイルを作成します。[こちら](https://gist.github.com/AlmostEfficient/669ac250214f30347097a1aeedcdfa12)からコピーしてください Solidity の文字列は少し特殊なので、変換するのに関数などが必要です。 この外部ライブラリは文字列をバイトに変換し、ガス効率を向上させます。

作成したスマートコントラクトは支払いを受ける準備ができています。

今、私たちのドメインを望んでいるすべての人に販売することができます。

`run.js`に向かい、次のように更新しましょう。

```jsx
const main = async () => {
  const domainContractFactory = await hre.ethers.getContractFactory("Domains");
  // "ninja"をデプロイ時にconstructorに渡します。
  const domainContract = await domainContractFactory.deploy("ninja");
  await domainContract.deployed();

  console.log("Contract deployed to:", domainContract.address);

  // valueで代金をやりとりしています。
  let txn = await domainContract.register("mortal", {
    value: hre.ethers.utils.parseEther("0.1"),
  });
  await txn.wait();

  const address = await domainContract.getAddress("mortal");
  console.log("Owner of domain mortal:", address);

  const balance = await hre.ethers.provider.getBalance(domainContract.address);
  console.log("Contract balance:", hre.ethers.utils.formatEther(balance));
};

const runMain = async () => {
  try {
    await main();
    process.exit(0);
  } catch (error) {
    console.log(error);
    process.exit(1);
  }
};

runMain();
```

ここでは実際に何をしているのでしょうか。

登録されているすべてのドメインの末尾を`.ninja`にします。 つまり、 `tld`プロパティを初期化するには、`deploy`関数に`ninja`を渡す必要があります。 次に、ドメインを登録するには、実際に`register`関数を呼び出す必要があります。 このとき 2 つの引数が必要であることに注意しておきましょう。

- 登録する domain
- **Matic 建ての domain（ガスを含む）の価格**

単位換算は[Solidity では動作が異なる](https://docs.soliditylang.org/en/v0.8.11/units-and-global-variables.html#units-and-globally-available)ため、特別な`parseEther`関数を使用します）。 これにより、支払いとして 0.1Matic がウォレットからコントラクトに送信されます。 その後、ドメインは私のウォレットアドレスに作成されます。

**たったこれだけです。**　

スマートコントラクトで支払いを行うのはとても簡単です。

高額な手数料を支払い処理業者に払ったり、クレジットカード手数料を払わなければならないといったこともありません。

数行のコードでいいのです。

<br/>

さあ、`run.js`スクリプトを実行しましょう。 これを行うと、次のような結果が表示されます。

![https://i.imgur.com/nLCRCKl.png](https://i.imgur.com/nLCRCKl.png)

**やりました！** シンプルなコントラクトコードだけで、ENS の基本的なアクションを実行できるようになりました。

ただし、これで終わりではありません。いくつか NFT を使用して、このドメインをもっとかっこよくしましょう 👀

### 💎Non Fungible ドメイン

ではドメインマッピングを**✨NFT✨**に変換してみましょう。

OpenSea に ENS ドメインを所有している場合、実際には次のようなものが表示されます。

![https://i.imgur.com/fs9TVN5.png](https://i.imgur.com/fs9TVN5.png)

もしかしたら、なぜ私たちは自分のドメインを NFT にする必要があるのだろうと考えるかもしれません。

ポイントは何なのでしょうか。

伝統的な web2 ドメインについて考えても、それらはすでに NFT とも言えます。

名前ごとにドメインは 1 つしか存在できません。

複数の人が同じドメインを所有することはできません。

ドメインを購入すると、実際にはインターネット全体で唯一のコピーを所有および管理します。

だから私たちはそれを活用してきたわけですが、これらを非常に簡単に交換/譲渡できればもっと素晴らしいのではないでしょうか。

結局、ENS ドメインは単なる"トークン"です。

<br/>

さらに進んで、NFT を使用してブロックチェーン上のドメインを最大限活用してみましょう 🤘

コードに取り組む前にここで実際に行うことを確認しましょう。

1. OpenZeppelin のコントラクトを使用して、ERC721 トークンを簡単に作成します。
2. NFT 用の SVG を作成し、チェーンストレージで使用します。
3. トークンメタデータ（ NFT が保持するデータ）を設定します。
4. ミントします！

最後には**新しく登録したドメインを取得して、そこから NFT を作成し**します。

コードを下のように変更します。`register`関数が特に大きく変更されています。あとでまた説明しますね。

```jsx
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.4;

// 最初にOpenZeppelinライブラリをインポートします.
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
import "@openzeppelin/contracts/utils/Counters.sol";

import {StringUtils} from "./libraries/StringUtils.sol";
// Base64のライブラリをインポートします。
import {Base64} from "./libraries/Base64.sol";

import "hardhat/console.sol";

// インポートしたコントラクトを継承します。継承したコントラクトのメソッドを使用できるようになります。
contract Domains is ERC721URIStorage {
  // OpenZeppelinによりtokenIdsの追跡が容易になります。
  using Counters for Counters.Counter;
  Counters.Counter private _tokenIds;

  string public tld;

  // NFTのイメージ画像をSVG形式でオンチェーンに保存します。
  string svgPartOne = '<svg xmlns="http://www.w3.org/2000/svg" width="270" height="270" fill="none"><path fill="url(#B)" d="M0 0h270v270H0z"/><defs><filter id="A" color-interpolation-filters="sRGB" filterUnits="userSpaceOnUse" height="270" width="270"><feDropShadow dx="0" dy="1" stdDeviation="2" flood-opacity=".225" width="200%" height="200%"/></filter></defs><path d="M72.863 42.949c-.668-.387-1.426-.59-2.197-.59s-1.529.204-2.197.59l-10.081 6.032-6.85 3.934-10.081 6.032c-.668.387-1.426.59-2.197.59s-1.529-.204-2.197-.59l-8.013-4.721a4.52 4.52 0 0 1-1.589-1.616c-.384-.665-.594-1.418-.608-2.187v-9.31c-.013-.775.185-1.538.572-2.208a4.25 4.25 0 0 1 1.625-1.595l7.884-4.59c.668-.387 1.426-.59 2.197-.59s1.529.204 2.197.59l7.884 4.59a4.52 4.52 0 0 1 1.589 1.616c.384.665.594 1.418.608 2.187v6.032l6.85-4.065v-6.032c.013-.775-.185-1.538-.572-2.208a4.25 4.25 0 0 0-1.625-1.595L41.456 24.59c-.668-.387-1.426-.59-2.197-.59s-1.529.204-2.197.59l-14.864 8.655a4.25 4.25 0 0 0-1.625 1.595c-.387.67-.585 1.434-.572 2.208v17.441c-.013.775.185 1.538.572 2.208a4.25 4.25 0 0 0 1.625 1.595l14.864 8.655c.668.387 1.426.59 2.197.59s1.529-.204 2.197-.59l10.081-5.901 6.85-4.065 10.081-5.901c.668-.387 1.426-.59 2.197-.59s1.529.204 2.197.59l7.884 4.59a4.52 4.52 0 0 1 1.589 1.616c.384.665.594 1.418.608 2.187v9.311c.013.775-.185 1.538-.572 2.208a4.25 4.25 0 0 1-1.625 1.595l-7.884 4.721c-.668.387-1.426.59-2.197.59s-1.529-.204-2.197-.59l-7.884-4.59a4.52 4.52 0 0 1-1.589-1.616c-.385-.665-.594-1.418-.608-2.187v-6.032l-6.85 4.065v6.032c-.013.775.185 1.538.572 2.208a4.25 4.25 0 0 0 1.625 1.595l14.864 8.655c.668.387 1.426.59 2.197.59s1.529-.204 2.197-.59l14.864-8.655c.657-.394 1.204-.95 1.589-1.616s.594-1.418.609-2.187V55.538c.013-.775-.185-1.538-.572-2.208a4.25 4.25 0 0 0-1.625-1.595l-14.993-8.786z" fill="#fff"/><defs><linearGradient id="B" x1="0" y1="0" x2="270" y2="270" gradientUnits="userSpaceOnUse"><stop stop-color="#cb5eee"/><stop offset="1" stop-color="#0cd7e4" stop-opacity=".99"/></linearGradient></defs><text x="32.5" y="231" font-size="27" fill="#fff" filter="url(#A)" font-family="Plus Jakarta Sans,DejaVu Sans,Noto Color Emoji,Apple Color Emoji,sans-serif" font-weight="bold">';
  string svgPartTwo = '</text></svg>';

  mapping(string => address) public domains;
  mapping(string => string) public records;

  constructor(string memory _tld) payable ERC721("Ninja Name Service", "NNS") {
    tld = _tld;
    console.log("%s name service deployed", _tld);
  }

  function register(string calldata name) public payable {
    require(domains[name] == address(0));

    uint256 _price = price(name);
    require(msg.value >= _price, "Not enough Matic paid");

    // ネームとTLD(トップレベルドメイン)を結合します。
    string memory _name = string(abi.encodePacked(name, ".", tld));
    // NFT用にSVGイメージを作成します。
    string memory finalSvg = string(abi.encodePacked(svgPartOne, _name, svgPartTwo));
    uint256 newRecordId = _tokenIds.current();
    uint256 length = StringUtils.strlen(name);
    string memory strLen = Strings.toString(length);

    console.log("Registering %s.%s on the contract with tokenID %d", name, tld, newRecordId);

    // JSON形式のNFTのメタデータを作成。文字列を結合しbase64でエンコードします。
    string memory json = Base64.encode(
      abi.encodePacked(
        '{"name": "',
        _name,
        '", "description": "A domain on the Ninja name service", "image": "data:image/svg+xml;base64,',
        Base64.encode(bytes(finalSvg)),
        '","length":"',
        strLen,
        '"}'
      )
    );

    string memory finalTokenUri = string( abi.encodePacked("data:application/json;base64,", json));

    console.log("\n--------------------------------------------------------");
    console.log("Final tokenURI", finalTokenUri);
    console.log("--------------------------------------------------------\n");

    _safeMint(msg.sender, newRecordId);
    _setTokenURI(newRecordId, finalTokenUri);
    domains[name] = msg.sender;

    _tokenIds.increment();
  }

  // price, getAddress, setRecord, getRecord などのfunction は変更しません。
}
```

_注：引き続き `price`、` getAddress`、`setRecord`および`getRecord`関数は必要です。変更されていないため、ここでは省略していますが消さないでください。_

```jsx
// 最初にOpenZeppelinライブラリをインポートします.
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
import "@openzeppelin/contracts/utils/Counters.sol";

// Base64のライブラリをインポートします。
import {Base64} from "./libraries/Base64.sol";

contract Domains is ERC721URIStorage {
```

前に出てきた`StringUtils`と同様に、既存の外部ライブラリをインポートしています。 今回は OpenZeppelin からインポートします。 次に、ドメインコントラクトを宣言するときに、`ERC721URIStorage`を使用してそれらを「継承」します。 継承について詳しくは[こちら](https://solidity-by-example.org/inheritance/)をご覧くださいが、基本的には、他のコントラクトを私たちから呼び出すことができることを意味します。 これは、私たちが使用する関数をインポートするようなものです。

では、ここから何が得られるのでしょうか。

`ERC721`として知られている規格を使用して NFT を作成することができます。

[こちら](https://eips.ethereum.org/EIPS/eip-721)で詳細を読むことができます。

OpenZeppelin は、NFT の標準を実装していて、それをベースに独自のロジックを記述してカスタマイズできるようにしています。 つまり、いわゆるボイラープレートコードを記述する必要はありません。 ライブラリを使用せずに HTTP サーバーを最初から作成するのは現実的ではないでしょう。 同様に、NFT コントラクトを最初から作成する必要はありません。

`Base64`に関しては、外部ライブラリの関数で、NFT イメージに使用される SVG とそのメタデータの JSON を Solidity の`Base64`に変換するのに役立ちます。 ライブラリフォルダ`libraries`に`Base64.sol`という名前のファイルを作成し、[ここ](https://gist.github.com/farzaa/f13f5d9bda13af68cc96b54851345832)からコードをコピーして貼り付けます。

```jsx
using Counters for Counters.Counter;
Counters.Counter private _tokenIds;
```

次に、コントラクトの上部に`Counters`が表示されます。これは何でしょうか。何かを数えるために必ずライブラリが必要なわけではありませんがここではガス効率が高く、NFT 用に構築されたものをインポートしています。

`_tokenIds`を使用して、NFT の一意の識別子を追跡します。 これは、`private _tokenIds`を宣言したときに自動的に 0 に初期化される数値です。

したがって、最初に `register`を呼び出すと、`newRecordId`は 0 になります。再度実行すると、 `newRecordId`は 1 になり、以下同様に続きます。 `_tokenIds`は**状態変数**であることに注意してください。これは、変更されると、値がコントラクトに直接保存されることを意味します。

```jsx
constructor(string memory _tld) payable ERC721("Ninja Name Service", "NNS") {
  tld = _tld;
  console.log("%s name service deployed", _tld);
}
```

ここで、継承したいすべての ERC721 コントラクト情報を取り込むようにコントラクトに指示しています。 渡す必要があるのは次のとおりです。

- NFT コレクション名、`Ninja Name Service`。
- NFT のシンボル名、`NNS`。

ご自分で好きな名前に変えてみてください。

次に、NFT を設計します。 これはアプリの非常に重要な部分となります。 NFT は、誰もが自分のウォレットや OpenSea で目にするものです。 実際には、NFT を Polygon ブロックチェーン上に保存します。 多くの NFT は、画像ホスティングサービスを指す単なるリンクです。 通常サーバーサービスがダウンした場合、NFT にはイメージ画像がなくなります。 それをブロックチェーン上に置くことによって、永続性を持ちます。

これを実現するために、SVG（コードで構築されたイメージ）を使用します。 グラデーション、ポリゴンロゴ、ドメインテキストを含む正方形のボックスの SVG コードは次のとおりです。

```html
<svg xmlns="http://www.w3.org/2000/svg" width="270" height="270" fill="none">
  <path fill="url(#B)" d="M0 0h270v270H0z" />
  <defs>
    <filter
      id="A"
      color-interpolation-filters="sRGB"
      filterUnits="userSpaceOnUse"
      height="270"
      width="270"
    >
      <feDropShadow
        dx="0"
        dy="1"
        stdDeviation="2"
        flood-opacity=".225"
        width="200%"
        height="200%"
      />
    </filter>
  </defs>
  <path
    d="M72.863 42.949c-.668-.387-1.426-.59-2.197-.59s-1.529.204-2.197.59l-10.081 6.032-6.85 3.934-10.081 6.032c-.668.387-1.426.59-2.197.59s-1.529-.204-2.197-.59l-8.013-4.721a4.52 4.52 0 0 1-1.589-1.616c-.384-.665-.594-1.418-.608-2.187v-9.31c-.013-.775.185-1.538.572-2.208a4.25 4.25 0 0 1 1.625-1.595l7.884-4.59c.668-.387 1.426-.59 2.197-.59s1.529.204 2.197.59l7.884 4.59a4.52 4.52 0 0 1 1.589 1.616c.384.665.594 1.418.608 2.187v6.032l6.85-4.065v-6.032c.013-.775-.185-1.538-.572-2.208a4.25 4.25 0 0 0-1.625-1.595L41.456 24.59c-.668-.387-1.426-.59-2.197-.59s-1.529.204-2.197.59l-14.864 8.655a4.25 4.25 0 0 0-1.625 1.595c-.387.67-.585 1.434-.572 2.208v17.441c-.013.775.185 1.538.572 2.208a4.25 4.25 0 0 0 1.625 1.595l14.864 8.655c.668.387 1.426.59 2.197.59s1.529-.204 2.197-.59l10.081-5.901 6.85-4.065 10.081-5.901c.668-.387 1.426-.59 2.197-.59s1.529.204 2.197.59l7.884 4.59a4.52 4.52 0 0 1 1.589 1.616c.384.665.594 1.418.608 2.187v9.311c.013.775-.185 1.538-.572 2.208a4.25 4.25 0 0 1-1.625 1.595l-7.884 4.721c-.668.387-1.426.59-2.197.59s-1.529-.204-2.197-.59l-7.884-4.59a4.52 4.52 0 0 1-1.589-1.616c-.385-.665-.594-1.418-.608-2.187v-6.032l-6.85 4.065v6.032c-.013.775.185 1.538.572 2.208a4.25 4.25 0 0 0 1.625 1.595l14.864 8.655c.668.387 1.426.59 2.197.59s1.529-.204 2.197-.59l14.864-8.655c.657-.394 1.204-.95 1.589-1.616s.594-1.418.609-2.187V55.538c.013-.775-.185-1.538-.572-2.208a4.25 4.25 0 0 0-1.625-1.595l-14.993-8.786z"
    fill="#fff"
  />
  <defs>
    <linearGradient
      id="B"
      x1="0"
      y1="0"
      x2="270"
      y2="270"
      gradientUnits="userSpaceOnUse"
    >
      <stop stop-color="#cb5eee" />
      <stop offset="1" stop-color="#0cd7e4" stop-opacity=".99" />
    </linearGradient>
  </defs>
  <text
    x="32.5"
    y="231"
    font-size="27"
    fill="#fff"
    filter="url(#A)"
    font-family="Plus Jakarta Sans,DejaVu Sans,Noto Color Emoji,Apple Color Emoji,sans-serif"
    font-weight="bold"
  >
    mortal.ninja
  </text>
</svg>
```

HTML ファイルのようなものですね。 ここでは SVG を作成する方法を知る必要はありません。 無料で作成できるツールはたくさんあります。 これは[Figma](https://www.figma.com/)を使って作りました。興味がある方は調べてみるといいでしょう。

[こちら](https://www.svgviewer.dev/)のウェブサイトにアクセスし、上のコードを貼り付けて確認してみてください。

**コード付きの画像**を作成できるので有用です。

SVG は**多くの場合**カスタマイズできます。SVG をアニメーション化することもできます。 詳細については、[こちら](https://developer.mozilla.org/en-US/docs/Web/SVG/Tutorial)をご参照ください。

最終的な NFT は以下のようなものになりました。

![https://i.imgur.com/epYuKfc.png](https://i.imgur.com/epYuKfc.png)

SVG をカスタマイズしてみても面白いでしょう。興味がある方は、アニメーション化された SVG を試すこともいいでしょう 👀

```jsx
  string svgPartOne = '<svg xmlns="http://www.w3.org/2000/svg" width="270" height="270" fill="none"><path fill="url(#B)" d="M0 0h270v270H0z"/><defs><filter id="A" color-interpolation-filters="sRGB" filterUnits="userSpaceOnUse" height="270" width="270"><feDropShadow dx="0" dy="1" stdDeviation="2" flood-opacity=".225" width="200%" height="200%"/></filter></defs><path d="M72.863 42.949c-.668-.387-1.426-.59-2.197-.59s-1.529.204-2.197.59l-10.081 6.032-6.85 3.934-10.081 6.032c-.668.387-1.426.59-2.197.59s-1.529-.204-2.197-.59l-8.013-4.721a4.52 4.52 0 0 1-1.589-1.616c-.384-.665-.594-1.418-.608-2.187v-9.31c-.013-.775.185-1.538.572-2.208a4.25 4.25 0 0 1 1.625-1.595l7.884-4.59c.668-.387 1.426-.59 2.197-.59s1.529.204 2.197.59l7.884 4.59a4.52 4.52 0 0 1 1.589 1.616c.384.665.594 1.418.608 2.187v6.032l6.85-4.065v-6.032c.013-.775-.185-1.538-.572-2.208a4.25 4.25 0 0 0-1.625-1.595L41.456 24.59c-.668-.387-1.426-.59-2.197-.59s-1.529.204-2.197.59l-14.864 8.655a4.25 4.25 0 0 0-1.625 1.595c-.387.67-.585 1.434-.572 2.208v17.441c-.013.775.185 1.538.572 2.208a4.25 4.25 0 0 0 1.625 1.595l14.864 8.655c.668.387 1.426.59 2.197.59s1.529-.204 2.197-.59l10.081-5.901 6.85-4.065 10.081-5.901c.668-.387 1.426-.59 2.197-.59s1.529.204 2.197.59l7.884 4.59a4.52 4.52 0 0 1 1.589 1.616c.384.665.594 1.418.608 2.187v9.311c.013.775-.185 1.538-.572 2.208a4.25 4.25 0 0 1-1.625 1.595l-7.884 4.721c-.668.387-1.426.59-2.197.59s-1.529-.204-2.197-.59l-7.884-4.59a4.52 4.52 0 0 1-1.589-1.616c-.385-.665-.594-1.418-.608-2.187v-6.032l-6.85 4.065v6.032c-.013.775.185 1.538.572 2.208a4.25 4.25 0 0 0 1.625 1.595l14.864 8.655c.668.387 1.426.59 2.197.59s1.529-.204 2.197-.59l14.864-8.655c.657-.394 1.204-.95 1.589-1.616s.594-1.418.609-2.187V55.538c.013-.775-.185-1.538-.572-2.208a4.25 4.25 0 0 0-1.625-1.595l-14.993-8.786z" fill="#fff"/><defs><linearGradient id="B" x1="0" y1="0" x2="270" y2="270" gradientUnits="userSpaceOnUse"><stop stop-color="#cb5eee"/><stop offset="1" stop-color="#0cd7e4" stop-opacity=".99"/></linearGradient></defs><text x="32.5" y="231" font-size="27" fill="#fff" filter="url(#A)" font-family="Plus Jakarta Sans,DejaVu Sans,Noto Color Emoji,Apple Color Emoji,sans-serif" font-weight="bold">';
  string svgPartTwo = '</text></svg>';
```

ここで行っているのは、ドメインに基づいて SVG を作成することだけです。 SVG を 2 つに分割し、その間にドメインを配置します。

```jsx
string memory _name = string(abi.encodePacked(name, ".", tld));
string memory finalSvg = string(abi.encodePacked(svgPartOne, _name, svgPartTwo));
```

この`abi.encodePacked`について簡単に見てみましょう。

Solidity の文字列が特殊だと言ったのを覚えていますでしょうか？ 

文字列を直接組み合わせることはできません。 

代わりに、`encodePacked`関数を使用して、一連の文字列をバイトに変換してから結合する必要があります。

```jsx
string(abi.encodePacked(svgPartOne, _name, svgPartTwo));
```

これは、SVG コードとドメインを実際に組み合わせる強力な 1 行です。`<svg>自分のドメイン</svg>`を実行するようなものです。

ドメインのアセットができたので、`register`関数を詳しく見て、メタデータがどのように構築されているかを確認しましょう。

```jsx
function register(string calldata name) public payable {
  require(domains[name] == address(0));

  uint256 _price = price(name);
  require(msg.value >= _price, "Not enough Matic paid");

  string memory _name = string(abi.encodePacked(name, ".", tld));
  string memory finalSvg = string(abi.encodePacked(svgPartOne, _name, svgPartTwo));
  uint256 newRecordId = _tokenIds.current();
  uint256 length = StringUtils.strlen(name);
  string memory strLen = Strings.toString(length);

  console.log("Registering %s on the contract with tokenID %d", name, newRecordId);

  string memory json = Base64.encode(
    abi.encodePacked(
        '{"name": "',
        _name,
        '", "description": "A domain on the Ninja name service", "image": "data:image/svg+xml;base64,',
        Base64.encode(bytes(finalSvg)),
        '","length":"',
        strLen,
        '"}'
    )
  );

  string memory finalTokenUri = string( abi.encodePacked("data:application/json;base64,", json));

  console.log("\n--------------------------------------------------------");
  console.log("Final tokenURI", finalTokenUri);
  console.log("--------------------------------------------------------\n");

  _safeMint(msg.sender, newRecordId);
  _setTokenURI(newRecordId, finalTokenUri);
  domains[name] = msg.sender;

  _tokenIds.increment();
}
```

ほとんどはもう学習済みです。今までで取り上げていないのは、`_tokenIds`と`json`の使用だけです。

`json` NFT は JSON を使用して、名前(name)、説明(description)、属性(attributes)、メディア(media)などの詳細情報を保存します。 `json`で行っているのは、文字列と`abi.encodePacked`を組み合わせて JSON オブジェクトを作成することです。 次に、トークンURI として設定する前に、Base64 文字列としてエンコードしています。

`_tokenIds`について知っておく必要があるのは、NFT の一意のトークン番号にアクセスして設定できるオブジェクトであるということだけです。 各 NFT には一意の`id`があり、それはNFTを確認するのに役立ちます。 以下の 2 つの行は、実際に NFT を作成する行です。

```jsx
// newRecordId にNFTをミントします。
_safeMint(msg.sender, newRecordId);

// ドメイン情報をnewRecordIdにセットします。
_setTokenURI(newRecordId, finalTokenUri);
```

`finalTokenUri`をコンソールに出力するコンソールログを追加しました。 `data：application / json; base64`の 1 つを取得し、それをブラウザのアドレスバーに入力すると、すべての JSON メタデータが表示されます。

### 🥸NFT ドメインをローカルで作成する

コントラクトを実行する準備ができました！ ローカルブロックチェーンでミントしてみましょう。

```
Compiled 13 Solidity files successfully
ninja name service deployed
Contract deployed to: 0x5FbDB2315678afecb367f032d93F642f64180aa3
Registering mortal.ninja on the contract with tokenID 0

--------------------------------------------------------
Final tokenURI data:application/json;base64,eyJuYW1lIjogIm1vcnRhbC5uaW5qYSIsICJkZXNjcmlwdGlvbiI6ICJBIGRvbWFpbiBvbiB0aGUgTmluamEgbmFtZSBzZXJ2aWNlIiwgImltYWdlIjogImRhdGE6aW1hZ2Uvc3ZnK3htbDtiYXNlNjQsUEhOMlp5QjRiV3h1Y3

//(長いため途中省略)

wWmlJZ1ptOXVkQzEzWldsbmFIUTlJbUp2YkdRaVBtMXZjblJoYkM1dWFXNXFZVHd2ZEdWNGRENDhMM04yWno0PSIsImxlbmd0aCI6IjYifQ==
--------------------------------------------------------

Owner of domain mortal: 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266
Contract balance: 0.1
```

`npx hardhat run scripts/run.js`を実行します。 大きな違いは、コンソールの出力です。 私の外観です（このスクリーンショットの URI は短縮してあります）：

![https://i.imgur.com/nOpI3oD.png](https://i.imgur.com/nOpI3oD.png)

`tokenURI`をコピーしてブラウザのアドレスに入力すると、JSON オブジェクトが表示されます。 別のタブで JSON オブジェクト内の image の部分のみを貼り付けると、NFT 画像が取得されます。

<br/>
つまり、

ブラウザのアドレスに下の様に入力すると JSON オブジェクトを表示できます。

```
data:application/json;base64,[ここにデコードしたいデータを入れるとブラウザ上にJSONオブジェクトが返ってきます]
```

JSON オブジェクトではなく image の画像を直接ブラウザに表示する場合、data:image/svg...の部分をブラウザのアドレスに入力します。

```
data:image/svg+xml;base64,[ここにデコードしたいデータを入れるとブラウザ上にimageが表示されます]
```

![https://i.imgur.com/UDQC0Wn.png](https://i.imgur.com/UDQC0Wn.png)

さぁ、ドメインサービスを作成することができましたね！

SVG をオンチェーンに生成もできました。

ポリゴン上での支払いも実装できました。

着実な進歩です！素晴らしいです！

<br/>

### 🙋‍♂️ 質問する

ここまでの作業で何かわからないことがある場合は、Discord の `#section-1` で質問をしてください。

ヘルプをするときのフローが円滑になるので、エラーレポートには下記の 3 点を記載してください ✨

```
1. 質問が関連しているセクション番号とレッスン番号
2. 何をしようとしていたか
3. エラー文をコピー&ペースト
4. エラー画面のスクリーンショット
```

---

お疲れ様でした。今回も長いレッスンでしたのでひと休みして次のレッスンに進んでくださいね 🎉
