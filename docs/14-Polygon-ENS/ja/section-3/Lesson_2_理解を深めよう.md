### 🦴すべてのドメインを取得

現在、アプリは確実に形になってきていますね。 コントラクトをさらに改善し、フロントエンド用に最適化するためにできることはまだあります。たとえば、既にミントされたドメインを取得する非常に簡単な方法があります。すべてのドメインを確認できます。

重要なのは、スマートコントラクトからこのデータを簡単に返すことができることです。
ブロックチェーン上の読み取りトランザクションは無料です🤑 だから、ガス代を払うことを心配する必要はありません！

このロジックを見て、これを行う方法を確認してください。

```solidity
// コントラクトの最初に付け加えてください（他のマッピングに続けて）。
mapping (uint => string) public names;

// コントラクトのどこかに付け加えてください。
function getAllNames() public view returns (string[] memory) {
  console.log("Getting all names from contract");
  string[] memory allNames = new string[](_tokenIds.current());
  for (uint i = 0; i < _tokenIds.current(); i++) {
    allNames[i] = names[i];
    console.log("Name for token %d is %s", i, allNames[i]);
  }

  return allNames;
}
```

とてもシンプルですね。 皆さん既にSolidityに習熟されているので理解しやすいはずです。

ドメイン名を持つミントIDを格納するためのマッピングと、それらを反復処理してリストに入れて送信するための`pure`関数を追加しました。 ただし、1つだけ欠けています。 マッピングデータを設定する必要があります。

これを`register`関数の最後の`_tokenIds.increment()`の直前に追加します。

```solidity
names[newRecordId] = name;
```

こうしてコントラクトで作成されたすべてのドメインを取得できます🤘

次のSectionでこの関数を実際に使用します。

### 💔コントラクトのドメインの有効性を確認

さて、おそらく「誰かが**長い**ドメインを作成しようとするとどうなりますか？ 👎」と考えられた方もいると思います。

素晴らしい疑問です。現在、フロントエンドはReactアプリでJavaScriptを使用して、ドメインが有効かどうかを確認しています。

ただ、誰かが私たちのコントラクトを直接使用して無効なドメインを作成する可能性があるため、これは最善のアイデアではありません。

下のように加えてみましょう。

```solidity
function valid(string calldata name) public pure returns(bool) {
  return StringUtils.strlen(name) >= 3 && StringUtils.strlen(name) <= 10;
}
```

Reactアプリで行っていたようにコントラクト側でドメイン名が3〜10文字であるかどうかを確認します。

無効な名前の場合は、`false`を返す必要があります。

### 🤬カスタムエラー

Solidityの最近のバージョンで追加された機能ですがカスタムエラーメッセージを使用できます。 これらは、エラーメッセージ文字列を繰り返す必要がないため、非常に便利です。デプロイ時にガスも節約できます。

この機能を使用するためにコントラクトのどこかに追加してください。

```solidity
error Unauthorized();
error AlreadyRegistered();
error InvalidName(string name);
```

<br>


```solidity
function setRecord(string calldata name, string calldata record) public {
  if (msg.sender != domains[name]) revert Unauthorized();
  records[name] = record;
}

function register(string calldata name) public payable {
  if (domains[name] != address(0)) revert AlreadyRegistered();
  if (!valid(name)) revert InvalidName(name);

  // register関数のその他の部分はそのまま残しておきます。
}
```

できました。

試しに長い文字列を登録しようとしたところ下のようなエラーが出ました。
(deploy.jsを元にした試行用のファイルを作成して使用した結果です。)

```
% npx hardhat run scripts/run_S3_L2.js
ninja name service deployed
Contract deployed to: 0xXXXXXXXXXXXXXXXXXXXXXXXXXXXX
Error: VM Exception while processing transaction: reverted with custom error 'InvalidName("banana_aaaaaaaaaaaaaaaaaaaa")'
```

このように機能を追加してデプロイすると、以前よりも多くのことを行うことができます。

### 🙋‍♂️ 質問する

ここまでの作業で何かわからないことがある場合は、Discord の `#section-3` で質問をしてください。

ヘルプをするときのフローが円滑になるので、エラーレポートには下記の 3 点を記載してください ✨

```
1. 質問が関連しているセクション番号とレッスン番号
2. 何をしようとしていたか
3. エラー文をコピー&ペースト
4. エラー画面のスクリーンショット
```

---
お疲れ様でした。一休みしてからでも次のセクションに進みましょう！！
