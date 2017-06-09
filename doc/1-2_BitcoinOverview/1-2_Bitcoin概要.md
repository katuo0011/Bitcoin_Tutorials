# 1-2. Bitcoin概要


## 1-2-1. 3つのネットワークを理解する

1. Mainnet - メインネットワーク
  * ``$ bitcoin-cli <コマンド>``  
  * 本番ネットワークである。
  * 実物のBitcoin(BTC)がやりとりされる。
2. Testnet - テストネットワーク
  * ``$ bitcoin-cli -testnet <コマンド>``
  * 準本番ネットワーク。
  * 本番リリース前の機能などがデプロイされてテストされている。基本的には、使える機能は Mainnet とほぼ同じで、より新しい機能を試せる環境である。
  * Mainnetと同様、マイニングが行われている。自分でマイニングしなくても基本的には誰かがマイニングしてくれている。
  * マイニングの難しさ(difficulty)は低く設定されている。
  * ネットワーク上のBTCは金銭的価値は無い。（※通貨としての価値を世の中の人が認めていないということ）
  * テスト環境であるため、よく壊れているので利用する際は注意。
3. Regtest - Regression Test用のネットワーク
  * ``$ bitcoin-cli -regtest <コマンド>``
  * ローカルテスト用のネットワーク。
  * 自分の端末上でのみ動く。
  * 自分でマイニングする必要がある。


## 1-2-2. Bitcoinのブロック、トランザクションの構造を理解する

### 1-2-2-1. ブロックの中身を見てみる
* 直近のblock情報(blockhash)を取得してみる。
* 直近のblockhashは``getblockchaininfo``で取得できる。``bestblockhash``でも同じ。
```
$ bitcoin-cli getbestblockhash
（※blockhashの情報が出力される。以下は例。）
00000000a823a457ebe54b7e5c6391b622b48b78926f6a93f84385671f34593c
```

* blockhashの情報から当該blockに記録されている情報を確認することができる。
```
$ bitcoin-cli getblock （※上で得たblockhashの情報）
{
  "hash": "00000000a823a457ebe54b7e5c6391b622b48b78926f6a93f84385671f34593c",
  "confirmations": 1,
  "strippedsize": 31239,
  "size": 31239,
  "weight": 124956,
  "height": 1125837,
  "version": 536870912,
  "versionHex": "20000000",
  "merkleroot": "9d74a62e4deff11fd45ea84611a6d6e7afbd9ebad2ca22e7f095926b5b77ed62",
  "tx": [
    "a7cdbc7aff3546da2f1db234fdbd730456eebb9e7cf98e480cf8207505c92429", 　※１
    "e1a4ba0821b962f10c4e1763f34d03a047786fc5a9e24b466f253925e698963c", 　※１
    （※中略）
    "31e39d629002848da87cdbd8f71df62cb6f6592a609e891721d4014439c87f36"　　※１
  ],
  "time": 1495945517,
  "mediantime": 1495937751,
  "nonce": 16708871,
  "bits": "1d00ffff",
  "difficulty": 1,
  "chainwork": "000000000000000000000000000000000000000000000024600d0f0621261bfd",
  "previousblockhash": "00000000d7de22f3146d326b06fe89e519d572f32f1417351fd05c63381ac3d4"　※２
}
```
* ※２は直前のblockhash。これが数珠つなぎになってチェーンを構成する。
* ※１はブロックに含まれるトランザクションの一覧である。

    * hash ⇒ the block hash
    * confirmations ⇒ The number of confirmations, or -1 if the block is not on the main chain
    * size ⇒ The block size
    * strippedsize ⇒ The block size excluding witness data
    * weight ⇒ The block weight as defined in BIP 141
    * height ⇒ The block height or index
    * version ⇒ The block version
    * versionHex ⇒ The block version formatted in hexadecimal
    * merkleroot ⇒ The merkle root
    * transactionid ⇒ The transaction id
    * time ⇒ The block time in seconds since epoch (Jan 1 1970 GMT)
    * mediantime ⇒ The median block time in seconds since epoch (Jan 1 1970 GMT)
    * nonce ⇒ The nonce
    * bits ⇒ The bits
    * difficulty ⇒ The difficulty
    * chainwork ⇒ Expected number of hashes required to produce the chain up to this block (in hex)
    * previousblockhash ⇒ The hash of the previous block
    * nextblockhash ⇒ The hash of the next block


### 1-2-2-2. ブロックに含まれるトランザクションの情報を見てみる
* 自身が発行したトランザクション以外に、他の人（他のクライアント）が発行して、ブロックに取り込まれたトランザクションの内容を確認することが可能である。
* 上記※１のトランザクションのうち``a7cdbc7aff3546da2f1db234fdbd730456eebb9e7cf98e480cf8207505c92429``のトランザクションの中身を見てみる。
* 他のクライアントが発行したトランザクションの内容を確認するには、まずRAWデータのトランザクション情報を取得（``getrawtransaction``）した後、decode（``decoderawtransaction``）する。
```
$ bitcoin-cli -testnet getrawtransaction 270e08dc06f353c55b7d6a250d82d6359af8ed3a332dfd332ee529ead2f09b57
01000000015a6649ea2c60dce4640321f33a67bbc8b19c7073eb8e050e9ee19866df78cf7a00000000fd6801004730440220627db2cfdc26b3ae28601eb65231f192ed2a413edc5388765897485617fdf3d702204c468d59e2f9f69952840db931ec036858b8d4e2e83ccafc80bb8a7fc320501101483045022100a8b141437a4eba780758f4844428b1890cf39efaa9df48d2bdb873ce081676e802204bfdbc67cf42588964a5dd0c61a83c41d5ac62f5f9e3681106f33ca99dfec56301483045022100d62ae28bd8d20e444500f517e6d47b6d183a80dcbc9a24dd7c220d76e6fc53c302206acc577a3c56f0b9ed9335bf83b7e3cceb1599251df5089fd483dcb6fbddc8f0014c8b532102bcdbcb324179b7648e2c7802da2c1a49ae94b51188d86e7352b39257f6bba56c2102fe22fdac89648502a6eeefd84b1616142e267efb1f54fa0932380ae2a14e764421031f1f73c3723efda2a31298729e56f69acfe0751c6520f69ff6f0941998634c462102fddf017fa0899ae0ea23cbd7f9b159b9d9fa8f56c62d55706913e407bb49125654aeffffffff022007d3050000000017a914716370a50fa600b4ecc34312b9d0d5f3c294166b870000000000000000326a3045584f4e554d010014d6150000000000f2b2585122992947106051af48fdbf59da3851aedd9c8775a96262fd70f804cd00000000
```
* トランザクション情報（RAWデータ）は上記の通り、Base58（https://ja.wikipedia.org/wiki/Base58）　というエンコード方式でエンコードされている。
* RAWデータをデコードは以下のように行う。
```
$ bitcoin-cli -testnet decoderawtransaction {※上で取得したRAWデータ}
{
  "txid": "270e08dc06f353c55b7d6a250d82d6359af8ed3a332dfd332ee529ead2f09b57",
  "hash": "270e08dc06f353c55b7d6a250d82d6359af8ed3a332dfd332ee529ead2f09b57",
  "version": 1,
  "size": 504,
  "vsize": 504,
  "locktime": 0,
  "vin": [
    {
      "txid": "7acf78df6698e19e0e058eeb73709cb1c8bb673af3210364e4dc602cea49665a",
      "vout": 0,
      "scriptSig": {
        "asm": "0 30440220627db2cfdc26b3ae28601eb65231f192ed2a413edc5388765897485617fdf3d702204c468d59e2f9f69952840db931ec036858b8d4e2e83ccafc80bb8a7fc3205011[ALL] 3045022100a8b141437a4eba780758f4844428b1890cf39efaa9df48d2bdb873ce081676e802204bfdbc67cf42588964a5dd0c61a83c41d5ac62f5f9e3681106f33ca99dfec563[ALL] 3045022100d62ae28bd8d20e444500f517e6d47b6d183a80dcbc9a24dd7c220d76e6fc53c302206acc577a3c56f0b9ed9335bf83b7e3cceb1599251df5089fd483dcb6fbddc8f0[ALL] 532102bcdbcb324179b7648e2c7802da2c1a49ae94b51188d86e7352b39257f6bba56c2102fe22fdac89648502a6eeefd84b1616142e267efb1f54fa0932380ae2a14e764421031f1f73c3723efda2a31298729e56f69acfe0751c6520f69ff6f0941998634c462102fddf017fa0899ae0ea23cbd7f9b159b9d9fa8f56c62d55706913e407bb49125654ae",
        "hex": "004730440220627db2cfdc26b3ae28601eb65231f192ed2a413edc5388765897485617fdf3d702204c468d59e2f9f69952840db931ec036858b8d4e2e83ccafc80bb8a7fc320501101483045022100a8b141437a4eba780758f4844428b1890cf39efaa9df48d2bdb873ce081676e802204bfdbc67cf42588964a5dd0c61a83c41d5ac62f5f9e3681106f33ca99dfec56301483045022100d62ae28bd8d20e444500f517e6d47b6d183a80dcbc9a24dd7c220d76e6fc53c302206acc577a3c56f0b9ed9335bf83b7e3cceb1599251df5089fd483dcb6fbddc8f0014c8b532102bcdbcb324179b7648e2c7802da2c1a49ae94b51188d86e7352b39257f6bba56c2102fe22fdac89648502a6eeefd84b1616142e267efb1f54fa0932380ae2a14e764421031f1f73c3723efda2a31298729e56f69acfe0751c6520f69ff6f0941998634c462102fddf017fa0899ae0ea23cbd7f9b159b9d9fa8f56c62d55706913e407bb49125654ae"
      },
      "sequence": 4294967295
    }
  ],
  "vout": [
    {
      "value": 0.97716,
      "n": 0,
      "scriptPubKey": {
        "asm": "OP_HASH160 716370a50fa600b4ecc34312b9d0d5f3c294166b OP_EQUAL",
        "hex": "a914716370a50fa600b4ecc34312b9d0d5f3c294166b87",
        "reqSigs": 1,
        "type": "scripthash",
        "addresses": [
          "2N3amUB4yLWS5dd1zMK3PisQ4UWHMgD6b7o"
        ]
      }
    },
    {
      "value": 0.00,
      "n": 1,
      "scriptPubKey": {
        "asm": "OP_RETURN 45584f4e554d010014d6150000000000f2b2585122992947106051af48fdbf59da3851aedd9c8775a96262fd70f804cd",
        "hex": "6a3045584f4e554d010014d6150000000000f2b2585122992947106051af48fdbf59da3851aedd9c8775a96262fd70f804cd",
        "type": "nulldata"
      }
    }
  ],
  "hex": "01000000015a6649ea2c60dce4640321f33a67bbc8b19c7073eb8e050e9ee19866df78cf7a00000000fd6801004730440220627db2cfdc26b3ae28601eb65231f192ed2a413edc5388765897485617fdf3d702204c468d59e2f9f69952840db931ec036858b8d4e2e83ccafc80bb8a7fc320501101483045022100a8b141437a4eba780758f4844428b1890cf39efaa9df48d2bdb873ce081676e802204bfdbc67cf42588964a5dd0c61a83c41d5ac62f5f9e3681106f33ca99dfec56301483045022100d62ae28bd8d20e444500f517e6d47b6d183a80dcbc9a24dd7c220d76e6fc53c302206acc577a3c56f0b9ed9335bf83b7e3cceb1599251df5089fd483dcb6fbddc8f0014c8b532102bcdbcb324179b7648e2c7802da2c1a49ae94b51188d86e7352b39257f6bba56c2102fe22fdac89648502a6eeefd84b1616142e267efb1f54fa0932380ae2a14e764421031f1f73c3723efda2a31298729e56f69acfe0751c6520f69ff6f0941998634c462102fddf017fa0899ae0ea23cbd7f9b159b9d9fa8f56c62d55706913e407bb49125654aeffffffff022007d3050000000017a914716370a50fa600b4ecc34312b9d0d5f3c294166b870000000000000000326a3045584f4e554d010014d6150000000000f2b2585122992947106051af48fdbf59da3851aedd9c8775a96262fd70f804cd00000000"
}
```
* ワンライナーで書くこともできて、その場合は ``bitcoin-cli -testnet decoderawtransaction $(bitcoin-cli -testnet getrawtransaction 270e08dc06f353c55b7d6a250d82d6359af8ed3a332dfd332ee529ead2f09b57)`` などとすれば良い。
* 各クライアントからトランザクションが生成され、ネットワーク（インターネット）にブロードキャストされる。他のクライアントがトランザクションを受け付けてmempoolに入れる。その後、マイニングされたトランザクションが上記の通りブロックに記録される。
* トランザクションデータの各項目の説明については、1.3にて後述する。


## 1-2-3. マイニング
* mempoolに入った複数のトランザクションをひとまとめにしたブロックをネットワークに提案する者が必要になる。
* マイニングアルゴリズム（Proof of Work）の詳細は後述するとして、ブロックの提案を行うのが『マイナー』と呼ばれるネットワーク参加者である。

### 1-2-3-1. マイニングの方法
* マイニングは``bitcoin-cli``を利用して、例えばテストネットでは以下の通り実行できる。
```
$ bitcoin-cli -testnet generate 1
```

* プロのマイニングは、「ASIC(Application Specific Integrated Circuit)」と呼ばれるBitcoinマイニング専用のICを利用してマイニングが行われる。有名なものにAntminer社のASICがある。
* ネットワークのハッシュパワーの情報などは以下のコマンドで確認できる。
```
$ bitcoin-cli -testnet getmininginfo
{
  "blocks": 1126198,
  "currentblocksize": 0,
  "currentblockweight": 0,
  "currentblocktx": 0,
  "difficulty": 3501386.391692169,
  "errors": "Warning: unknown new rules activated (versionbit 28)",
  "networkhashps": 9966245070079.24,
  "pooledtx": 44,
  "chain": "test"
}
```

  * "blocks": nnn　⇒　The current block
  * "currentblocksize": nnn　⇒　The last block size
  * "currentblockweight": nnn　⇒　The last block weight
  * "currentblocktx": nnn　⇒　The last block transaction
  * "difficulty": xxx.xxxxx　⇒　The current difficulty
  * "errors": "..."　⇒　Current errors
  * "networkhashps": nnn　⇒　The network hashes per second
  * "pooledtx": n　⇒　The size of the mempool
  * "chain": "xxxx"　⇒　current network name as defined in BIP70 (main, test, regtest)

### 1-2-3-2. マイニングのインセンティブ
* 後述するマイニング計算（Proof of Work）は、膨大な量のハッシュ計算を行うため、インフラ投資と電気使用料が必要になる。
* マイニングを行う目的は、マイニングに成功することによって得られる報酬にある。

  1. Block Reward(報酬)
    * 現在は12.5 BTC
    * このRewardは、Coinbaseのトランザクションになる。CoinbaseとはTxInが無いトランザクションである。
  2. Transaction Fee (手数料)
    * 提案するブロックに含める全トランザクションの手数料を総取りできる。
    * Block Reward は4年毎に半減し2140年頃ゼロになる。

* 大量のインフラ投資を行ってネットワークの計算力を乗っ取りビットコインを盗むよりも、マイニングの報酬を得るほうが容易である。経済合理性でネットワークが健全な状態に保たれている。
* また、かけた電気代にマイニングという付加価値を加えることによって報酬（売上）が得られるという構造は、経済的な不平等性があまりない。

### 1-2-3-3. Proof of Work (PoW)
* Hashcash (Adam Back, 1997)
* https://en.wikipedia.org/wiki/Hashcash
* BTCのMainnetでは1ブロック ⇒ 10分程度にコントロールされている。
* nonceを生成⇒ブロックのハッシュ値を作成⇒結果が『ターゲット』より小さな値であればブロックをブロードキャストする。
* 例）　``0x0000000000000000029d053e0ebe6a40db0fb47c5b24e30dbd751f0d767cfcda``　<　``0x00000000FFFF0000000000000000000000000000000000000000000000000000``
* ターゲットは2016ブロックごとに再設定される(2週間分)
