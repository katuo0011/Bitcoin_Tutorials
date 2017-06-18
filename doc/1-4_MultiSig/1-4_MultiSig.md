# 1-4. マルチシグネチャ（MultiSig）
* 参考１：　https://bitcoin.org/en/developer-examples#p2sh-multisig
* 参考２：　http://gaiax-blockchain.com/escrow

## 1-4-1. MultiSigとは
### 1-4-1-1. 基本知識
* 通常のアドレス
  * 鍵が1個かけられている。
  * 送金するには対応した鍵が１個必要。
* マルチシグアドレス
  * 鍵がM個かけられている。
  * 送金するにはそのうちN個の鍵で解錠する。（N ≦ M）
* 基本用語
  * m-of-n
    * m: minimum number of signature: 最低必要な署名の数
    * n: number of public key: 公開鍵の数
  * P2PKH
    * Pay To Public Key Hashの略。
    * 本番ネットの場合、1ではじまるBitcoinアドレス。
  * P2SH
    * Pay To Script Hashの略。
    * 本番ネットの場合、3ではじまるBitcoinアドレス。
    * アドレス・フォーマットについては、BIP13を参照。
  * redeemScript (serialized script)
    * 2of3の場合、3つの公開鍵から2つの秘密鍵チェックするScriptをハッシュ化したものを指す。
* その他
  * カラードコイン（Counterpartyなど）でも利用可能である。

### 1-4-1-2. MultiSigでどのようなことができるのか？
* 例①：商品の売買における安全な支払い
  * 2 of 3 のMultiSigなどで実装する。
  * 仲介機関として、MultiSigアドレスを利用する。
  * 補足資料（Escrowイメージ図）
* 例②：コインの固定化
  * 2 of 2 のMultiSigなどで実装する。
  * サービス提供者（取引所、金融機関、ポイント管理会社など）で、コインの流通を管理する用途。
  * コイン保持者とサービス提供者のMultiSigでコイン残高を管理することにより、サービス外で自由に流通することを防ぐことができる。
  * マネーロンダリングの防止や、ハッキングの防止などの目的で利用する場合が多い。
* 例③：投票
  * n of m (n ≦ m)のMultiSigなどで実装する。
  * m人の参加者のうちn人が署名した場合に、ある提案（トランザクション）が有効になる、といった用途。


## 1-4-2. MultiSigを試してみる
2 of 3 の MultiSig を試してみる。

### 1-4-2-1. P2PKHアドレスを作成する
* MultiSigを行うために必要なアドレス（P2PKHアドレス）を新規で発行する。（2of3の3つのアドレス）
```
$ bitcoin-cli -testnet getnewaddress
mweghCpKecju9ScXBNnSRFATY75HGiSkVi
$ bitcoin-cli -testnet getnewaddress
mpG9dJjWWK1sQqhtoVP5PgYZy73DLpkUkM
$ bitcoin-cli -testnet getnewaddress
myABybjVWKJhxg9Hs5ZsLfNhngSxgAvdc9
```

* 作成したアドレスは後から使いやすいように変数に格納する。
```
$ export NEW_ADDRESS1=mweghCpKecju9ScXBNnSRFATY75HGiSkVi
$ echo $NEW_ADDRESS1
mweghCpKecju9ScXBNnSRFATY75HGiSkVi
$ export NEW_ADDRESS2=mpG9dJjWWK1sQqhtoVP5PgYZy73DLpkUkM
$ echo $NEW_ADDRESS2
mpG9dJjWWK1sQqhtoVP5PgYZy73DLpkUkM
$ export NEW_ADDRESS3=myABybjVWKJhxg9Hs5ZsLfNhngSxgAvdc9
$ echo $NEW_ADDRESS3
myABybjVWKJhxg9Hs5ZsLfNhngSxgAvdc9
```

* MultiSig アドレスは通常 public key をハッシュ化して作られるBitcoinアドレスを利用して作成するが、 public key そのものも利用できる。
* ここではアドレス３のみ public key を使うことにする。
* public key はアドレスを作成したノード（※アドレス→公開鍵の変換は不可逆なのでアドレスを作成したノードしか知り得ない）で``validateaddress``メソッドを用いて表示することができる。
```
$ bitcoin-cli -testnet validateaddress $NEW_ADDRESS3
{
  "isvalid": true,
  "address": "myABybjVWKJhxg9Hs5ZsLfNhngSxgAvdc9",
  "scriptPubKey": "76a914c184f32def5ad56b5e4fc54c0b40e55abea8a4a288ac",
  "ismine": true,
  "iswatchonly": false,
  "isscript": false,
  "pubkey": "0346c274aca653ba608b4163cd41b6b66f584d3e2e772bb349c18f9e6aff78eaba",
  "iscompressed": true,
  "account": "",
  "timestamp": 1495421905,
  "hdkeypath": "m/0'/0'/5'",
  "hdmasterkeyid": "b7a508c102be6c42abf72aff33b334bd487f2164"
}
```
```
$ export NEW_ADDRESS3_PUBLIC_KEY=0346c274aca653ba608b4163cd41b6b66f584d3e2e772bb349c18f9e6aff78eaba
$ echo $NEW_ADDRESS3_PUBLIC_KEY
0346c274aca653ba608b4163cd41b6b66f584d3e2e772bb349c18f9e6aff78eaba
```

### 1-4-2-2. MultiSigアドレス（P2SH）を作成する
* 仲介者にあたる、MultiSigアドレス（P2SHアドレス）を作成する。
* ``createmultisig``メソッドを実行して、P2SHアドレスを作成する。
```
[IN]
$ bitcoin-cli -testnet createmultisig 2 '''[
  "'$NEW_ADDRESS1'",
  "'$NEW_ADDRESS2'",
  "'$NEW_ADDRESS3_PUBLIC_KEY'"
  ]'''
```
```
[OUT]
{
  "address": "2MwLGvwmtMEULetku9Es4fRzFeFM9E9tpWf",
  "redeemScript": "52210314d9fbcd2d1afa935eecfe9faa3911b55c3d0dd3e0253dd5431f397a4a67e2d32102c17be5afd7837b6aad634b59113b646785b323117c562bf7ba70bf988164bf68210346c274aca653ba608b4163cd41b6b66f584d3e2e772bb349c18f9e6aff78eaba53ae"
}
```

* 後ほど必要になってくる変数を保存しておく。
```
$ export P2SH_ADDRESS=2MwLGvwmtMEULetku9Es4fRzFeFM9E9tpWf
$ echo $P2SH_ADDRESS
2MwLGvwmtMEULetku9Es4fRzFeFM9E9tpWf
```
```
$ export P2SH_REDEEM_SCRIPT=52210314d9fbcd2d1afa935eecfe9faa3911b55c3d0dd3e0253dd5431f397a4a67e2d32102c17be5afd7837b6aad634b59113b646785b323117c562bf7ba70bf988164bf68210346c274aca653ba608b4163cd41b6b66f584d3e2e772bb349c18f9e6aff78eaba53ae
$ echo $P2SH_REDEEM_SCRIPT
52210314d9fbcd2d1afa935eecfe9faa3911b55c3d0dd3e0253dd5431f397a4a67e2d32102c17be5afd7837b6aad634b59113b646785b323117c562bf7ba70bf988164bf68210346c274aca653ba608b4163cd41b6b66f584d3e2e772bb349c18f9e6aff78eaba53ae
```

### 1-4-2-3. MultiSigアドレス向けにBTCを送金する
* 現在の残高を確認する。
```
$ bitcoin-cli -testnet getbalance
1.08440581
```

* MultiSigアドレス（P2SH_ADDRESS）に向けて0.1BTC送金する。
```
$ bitcoin-cli -testnet sendtoaddress $P2SH_ADDRESS 0.1
6d290b9beb5c0e18e13ba2939468416a2d2ac8c823673c094bfbffa2f0268c51
```

* MultiSigアドレスからの出金を行うには、上記の返り値であるトランザクションID(TXID)が必要になる。変数を保存しておく。
```
$ export UTXO_TXID=6d290b9beb5c0e18e13ba2939468416a2d2ac8c823673c094bfbffa2f0268c51
$ echo $UTXO_TXID
d31967022b8405351bc7eb53dfc215722500d5044dc355159d60e3ac0e9006f7
```

* 送金後の残高を確認する。送金した0.1BTCと手数料分が減っていることが確認できる。
```
$ bitcoin-cli -testnet getbalance
0.98439826
```

* 送金のトランザクション（MultiSigアドレスが保持しているトランザクション）はTXIDを指定して、以下のように取得できる。（※以下は、送金直後のトランザクションの中身。``blockhash``などの項目は、ブロックに取り込まれた時点で記録される。）
```
$ bitcoin-cli -testnet getrawtransaction $UTXO_TXID 1
{
  "hex": "0200000001f706900eace3609d1555c34d04d500257215c2df53ebc71b3505842b026719d3010000006a4730440220712bd0a5ad9cb0c18bfa4a93c9d047b97d0f074dd7fcf9537209c88350337101022038ceec727527d4f01f1b30f6d7e1d44d27dd54e0e84cb5ce2c2d6019259b1f340121025c351726ef30ba51e896b32615dbcf1c580c4cb36ff3ef838ed8b088fe6fdf69feffffff025205db05000000001976a914a1583b5cac7de6aacb4f41c2e3f49e03adb613ec88ac809698000000000017a9142cd50a06bc2b9b1015a1af7d2aba24aeae7d4b3587c72f1100",
  "txid": "6d290b9beb5c0e18e13ba2939468416a2d2ac8c823673c094bfbffa2f0268c51",
  "hash": "6d290b9beb5c0e18e13ba2939468416a2d2ac8c823673c094bfbffa2f0268c51",
  "version": 2,
  "size": 223,
  "vsize": 223,
  "locktime": 1126343,
  "vin": [
    {
      "txid": "d31967022b8405351bc7eb53dfc215722500d5044dc355159d60e3ac0e9006f7",
      "vout": 1,
      "scriptSig": {
        "asm": "30440220712bd0a5ad9cb0c18bfa4a93c9d047b97d0f074dd7fcf9537209c88350337101022038ceec727527d4f01f1b30f6d7e1d44d27dd54e0e84cb5ce2c2d6019259b1f34[ALL] 025c351726ef30ba51e896b32615dbcf1c580c4cb36ff3ef838ed8b088fe6fdf69",
        "hex": "4730440220712bd0a5ad9cb0c18bfa4a93c9d047b97d0f074dd7fcf9537209c88350337101022038ceec727527d4f01f1b30f6d7e1d44d27dd54e0e84cb5ce2c2d6019259b1f340121025c351726ef30ba51e896b32615dbcf1c580c4cb36ff3ef838ed8b088fe6fdf69"
      },
      "sequence": 4294967294
    }
  ],
  "vout": [
    {
      "value": 0.98239826,
      "n": 0,
      "scriptPubKey": {
        "asm": "OP_DUP OP_HASH160 a1583b5cac7de6aacb4f41c2e3f49e03adb613ec OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "76a914a1583b5cac7de6aacb4f41c2e3f49e03adb613ec88ac",
        "reqSigs": 1,
        "type": "pubkeyhash",
        "addresses": [
          "mvE4nQ19bs6m37WSoCQVxxvHmRzpfDvSPk"
        ]
      }
    },
    {
      "value": 0.10,
      "n": 1,
      "scriptPubKey": {
        "asm": "OP_HASH160 2cd50a06bc2b9b1015a1af7d2aba24aeae7d4b35 OP_EQUAL",
        "hex": "a9142cd50a06bc2b9b1015a1af7d2aba24aeae7d4b3587",
        "reqSigs": 1,
        "type": "scripthash",
        "addresses": [
          "2MwLGvwmtMEULetku9Es4fRzFeFM9E9tpWf"
        ]
      }
    }
  ],
  "hex": "0200000001f706900eace3609d1555c34d04d500257215c2df53ebc71b3505842b026719d3010000006a4730440220712bd0a5ad9cb0c18bfa4a93c9d047b97d0f074dd7fcf9537209c88350337101022038ceec727527d4f01f1b30f6d7e1d44d27dd54e0e84cb5ce2c2d6019259b1f340121025c351726ef30ba51e896b32615dbcf1c580c4cb36ff3ef838ed8b088fe6fdf69feffffff025205db05000000001976a914a1583b5cac7de6aacb4f41c2e3f49e03adb613ec88ac809698000000000017a9142cd50a06bc2b9b1015a1af7d2aba24aeae7d4b3587c72f1100"
}
```

* 上記の場合、アドレス``2MwLGvwmtMEULetku9Es4fRzFeFM9E9tpWf``宛のvoutがMultiSigアドレス宛のtxoutである。
* ``mvE4nQ19bs6m37WSoCQVxxvHmRzpfDvSPk``宛のvoutは送金者への戻りのtxoutである。

* MultiSigアドレスからの出金時にoutputの番号（voutの``n``）と、scriptPubKey（voutの``hex``）を指定する必要がある。これらを変数に格納する。
```
$ export UTXO_VOUT=1
$ echo $UTXO_VOUT
1
$ export UTXO_OUTPUT_SCRIPT=a9142cd50a06bc2b9b1015a1af7d2aba24aeae7d4b3587
$ echo $UTXO_OUTPUT_SCRIPT
a9142cd50a06bc2b9b1015a1af7d2aba24aeae7d4b3587
```

### 1-4-2-4. MultiSigアドレスから出金
* ここでは、出金先のアドレスを新しく作成することにする。
```
$ bitcoin-cli -testnet getnewaddress
mpJ5aVwC1BmVgHMVccsHdGTJNmh6u4XB1K
```

* 新しく作成したアドレスを変数に格納する。
```
$ export NEW_ADDRESS4=mpJ5aVwC1BmVgHMVccsHdGTJNmh6u4XB1K
$ echo $NEW_ADDRESS4
mpJ5aVwC1BmVgHMVccsHdGTJNmh6u4XB1K
```

* MultiSigアドレスから出金を行う。出金用のトランザクション（RAWデータ）を生成する。
* 以下の例は、MultiSigアドレスから出金を行い、アドレス４（``mpJ5aVwC1BmVgHMVccsHdGTJNmh6u4XB1K``）に0.03BTCを送るためのトランザクションである。
```
[IN]
$ bitcoin-cli -testnet createrawtransaction '''[
  {
    "txid": "'$UTXO_TXID'",
    "vout": '$UTXO_VOUT'
  }
  ]
  ''' '''
  {
    "'$NEW_ADDRESS4'": 0.03
  }'''
```
```
[OUT]
0200000001518c26f0a2fffb4b093c6723c8c82a2d6a41689493a23be1180e5ceb9b0b296d0100000000ffffffff01c0c62d00000000001976a9146049cdf6e6fc0151adad5f7200ebd3f61ee1fa9b88ac00000000
```

* 後から使いやすいように、得られたトランザクションRAWデータを変数に格納しておく。
```
$ export RAW_TX=0200000001518c26f0a2fffb4b093c6723c8c82a2d6a41689493a23be1180e5ceb9b0b296d0100000000ffffffff01c0c62d00000000001976a9146049cdf6e6fc0151adad5f7200ebd3f61ee1fa9b88ac00000000
$ echo $RAW_TX
0200000001518c26f0a2fffb4b093c6723c8c82a2d6a41689493a23be1180e5ceb9b0b296d0100000000ffffffff01c0c62d00000000001976a9146049cdf6e6fc0151adad5f7200ebd3f61ee1fa9b88ac00000000
```

* （おまけ）生成されたトランザクションをデコードして見てみる。
```
$ bitcoin-cli -testnet decoderawtransaction $RAW_TX
{
  "txid": "e631e7e1deeff8fb62212ce856914f84885998e6f82a3804bd93736a5109361c",
  "hash": "e631e7e1deeff8fb62212ce856914f84885998e6f82a3804bd93736a5109361c",
  "version": 2,
  "size": 85,
  "vsize": 85,
  "locktime": 0,
  "vin": [
    {
      "txid": "6d290b9beb5c0e18e13ba2939468416a2d2ac8c823673c094bfbffa2f0268c51",
      "vout": 1,
      "scriptSig": {
        "asm": "",
        "hex": ""
      },
      "sequence": 4294967295
    }
  ],
  "vout": [
    {
      "value": 0.03,
      "n": 0,
      "scriptPubKey": {
        "asm": "OP_DUP OP_HASH160 6049cdf6e6fc0151adad5f7200ebd3f61ee1fa9b OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "76a9146049cdf6e6fc0151adad5f7200ebd3f61ee1fa9b88ac",
        "reqSigs": 1,
        "type": "pubkeyhash",
        "addresses": [
          "mpJ5aVwC1BmVgHMVccsHdGTJNmh6u4XB1K"
        ]
      }
    }
  ],
  "hex": "0200000001518c26f0a2fffb4b093c6723c8c82a2d6a41689493a23be1180e5ceb9b0b296d0100000000ffffffff01c0c62d00000000001976a9146049cdf6e6fc0151adad5f7200ebd3f61ee1fa9b88ac00000000"
}
```

* 署名を行うために秘密鍵が必要になるので、それぞれのアドレスに対応する秘密鍵を取得し、変数に格納する。（※下記は念の為情報は伏せている）
* 2 of 3 の署名に対して、ここではアドレス１と３の秘密鍵を利用することとする。
```
$ bitcoin-cli -testnet dumpprivkey $NEW_ADDRESS1
{秘密鍵１}
$ bitcoin-cli -testnet dumpprivkey $NEW_ADDRESS3
cVTxc34WBk7mCy7BGAd3mNyfcGxijf6VG4p3xua1aSxtTZQNqYBo
｛秘密鍵３｝
$ export NEW_ADDRESS1_PRIVATE_KEY=｛秘密鍵１｝
$ export NEW_ADDRESS3_PRIVATE_KEY=｛秘密鍵３｝
```

* 秘密鍵１で1つ目の署名を行う。
* Redeem Script と アドレス１の private key の情報が必要になる。
```
[IN]
$ bitcoin-cli -testnet signrawtransaction $RAW_TX '''
    [
      {
        "txid": "'$UTXO_TXID'",
        "vout": '$UTXO_VOUT',
        "scriptPubKey": "'$UTXO_OUTPUT_SCRIPT'",
        "redeemScript": "'$P2SH_REDEEM_SCRIPT'"
      }
    ]
    ''' '''
    [
      "'$NEW_ADDRESS1_PRIVATE_KEY'"
    ]'''
```
```
[OUT]
{
  "hex": "0200000001518c26f0a2fffb4b093c6723c8c82a2d6a41689493a23be1180e5ceb9b0b296d01000000b50048304502210093427236935cddad3afbe5a13ed583ac31e8407ca694fb7a8dc3c2785b91c70b022061a1c0c91ec46e53b726c5d54fd1b40e92c0152d8c676be7e606eff7dbf81423014c6952210314d9fbcd2d1afa935eecfe9faa3911b55c3d0dd3e0253dd5431f397a4a67e2d32102c17be5afd7837b6aad634b59113b646785b323117c562bf7ba70bf988164bf68210346c274aca653ba608b4163cd41b6b66f584d3e2e772bb349c18f9e6aff78eaba53aeffffffff01c0c62d00000000001976a9146049cdf6e6fc0151adad5f7200ebd3f61ee1fa9b88ac00000000",
  "complete": false,
  "errors": [
    {
      "txid": "6d290b9beb5c0e18e13ba2939468416a2d2ac8c823673c094bfbffa2f0268c51",
      "vout": 1,
      "scriptSig": "0048304502210093427236935cddad3afbe5a13ed583ac31e8407ca694fb7a8dc3c2785b91c70b022061a1c0c91ec46e53b726c5d54fd1b40e92c0152d8c676be7e606eff7dbf81423014c6952210314d9fbcd2d1afa935eecfe9faa3911b55c3d0dd3e0253dd5431f397a4a67e2d32102c17be5afd7837b6aad634b59113b646785b323117c562bf7ba70bf988164bf68210346c274aca653ba608b4163cd41b6b66f584d3e2e772bb349c18f9e6aff78eaba53ae",
      "sequence": 4294967295,
      "error": "Operation not valid with the current stack size"
    }
  ]
}
```
* 2 of 3 のMultiSigであるため、1つ目の署名を行った時点では、``"complete": false``の状態になる。
* hex(トランザクションのRAWデータ)が署名前と変わっていることが見て取れるはずである。

* １つめの署名後のhexを変数に格納する。
```
$ export PARTLY_SIGNED_RAW_TX=0200000001518c26f0a2fffb4b093c6723c8c82a2d6a41689493a23be1180e5ceb9b0b296d01000000b50048304502210093427236935cddad3afbe5a13ed583ac31e8407ca694fb7a8dc3c2785b91c70b022061a1c0c91ec46e53b726c5d54fd1b40e92c0152d8c676be7e606eff7dbf81423014c6952210314d9fbcd2d1afa935eecfe9faa3911b55c3d0dd3e0253dd5431f397a4a67e2d32102c17be5afd7837b6aad634b59113b646785b323117c562bf7ba70bf988164bf68210346c274aca653ba608b4163cd41b6b66f584d3e2e772bb349c18f9e6aff78eaba53aeffffffff01c0c62d00000000001976a9146049cdf6e6fc0151adad5f7200ebd3f61ee1fa9b88ac00000000
$ echo $PARTLY_SIGNED_RAW_TX
0200000001518c26f0a2fffb4b093c6723c8c82a2d6a41689493a23be1180e5ceb9b0b296d01000000b50048304502210093427236935cddad3afbe5a13ed583ac31e8407ca694fb7a8dc3c2785b91c70b022061a1c0c91ec46e53b726c5d54fd1b40e92c0152d8c676be7e606eff7dbf81423014c6952210314d9fbcd2d1afa935eecfe9faa3911b55c3d0dd3e0253dd5431f397a4a67e2d32102c17be5afd7837b6aad634b59113b646785b323117c562bf7ba70bf988164bf68210346c274aca653ba608b4163cd41b6b66f584d3e2e772bb349c18f9e6aff78eaba53aeffffffff01c0c62d00000000001976a9146049cdf6e6fc0151adad5f7200ebd3f61ee1fa9b88ac00000000
```

* 2つ目の署名を行う。
* ここでも Redeem Script と、アドレス３の private key が必要になる。
```
[IN]
$ bitcoin-cli -testnet signrawtransaction $PARTLY_SIGNED_RAW_TX '''
    [
      {
        "txid": "'$UTXO_TXID'",
        "vout": '$UTXO_VOUT',
        "scriptPubKey": "'$UTXO_OUTPUT_SCRIPT'",
        "redeemScript": "'$P2SH_REDEEM_SCRIPT'"
      }
    ]
    ''' '''
    [
      "'$NEW_ADDRESS3_PRIVATE_KEY'"
    ]'''
```
```
[OUT]
{
  "hex": "0200000001518c26f0a2fffb4b093c6723c8c82a2d6a41689493a23be1180e5ceb9b0b296d01000000fdfe000048304502210093427236935cddad3afbe5a13ed583ac31e8407ca694fb7a8dc3c2785b91c70b022061a1c0c91ec46e53b726c5d54fd1b40e92c0152d8c676be7e606eff7dbf8142301483045022100898ef207e8a56d415bb3e4a5e9b7c3033dc58c06947efd655eeba48a29ed7632022044808c589a2905fd3edaebf3d844e8ef7894aa833aea314f5248838982657d55014c6952210314d9fbcd2d1afa935eecfe9faa3911b55c3d0dd3e0253dd5431f397a4a67e2d32102c17be5afd7837b6aad634b59113b646785b323117c562bf7ba70bf988164bf68210346c274aca653ba608b4163cd41b6b66f584d3e2e772bb349c18f9e6aff78eaba53aeffffffff01c0c62d00000000001976a9146049cdf6e6fc0151adad5f7200ebd3f61ee1fa9b88ac00000000",
  "complete": true
}
```
* 2 of 3のMultiSigの2つ目の署名が成功することで、``"complete": true``となることが見て取れるはずである。

* 署名済みのトランザクション（hex）を変数に格納する
```
$ export  SIGNED_RAW_TX=0200000001518c26f0a2fffb4b093c6723c8c82a2d6a41689493a23be1180e5ceb9b0b296d01000000fdfe000048304502210093427236935cddad3afbe5a13ed583ac31e8407ca694fb7a8dc3c2785b91c70b022061a1c0c91ec46e53b726c5d54fd1b40e92c0152d8c676be7e606eff7dbf8142301483045022100898ef207e8a56d415bb3e4a5e9b7c3033dc58c06947efd655eeba48a29ed7632022044808c589a2905fd3edaebf3d844e8ef7894aa833aea314f5248838982657d55014c6952210314d9fbcd2d1afa935eecfe9faa3911b55c3d0dd3e0253dd5431f397a4a67e2d32102c17be5afd7837b6aad634b59113b646785b323117c562bf7ba70bf988164bf68210346c274aca653ba608b4163cd41b6b66f584d3e2e772bb349c18f9e6aff78eaba53aeffffffff01c0c62d00000000001976a9146049cdf6e6fc0151adad5f7200ebd3f61ee1fa9b88ac00000000
$ echo $SIGNED_RAW_TX
0200000001518c26f0a2fffb4b093c6723c8c82a2d6a41689493a23be1180e5ceb9b0b296d01000000fdfe000048304502210093427236935cddad3afbe5a13ed583ac31e8407ca694fb7a8dc3c2785b91c70b022061a1c0c91ec46e53b726c5d54fd1b40e92c0152d8c676be7e606eff7dbf8142301483045022100898ef207e8a56d415bb3e4a5e9b7c3033dc58c06947efd655eeba48a29ed7632022044808c589a2905fd3edaebf3d844e8ef7894aa833aea314f5248838982657d55014c6952210314d9fbcd2d1afa935eecfe9faa3911b55c3d0dd3e0253dd5431f397a4a67e2d32102c17be5afd7837b6aad634b59113b646785b323117c562bf7ba70bf988164bf68210346c274aca653ba608b4163cd41b6b66f584d3e2e772bb349c18f9e6aff78eaba53aeffffffff01c0c62d00000000001976a9146049cdf6e6fc0151adad5f7200ebd3f61ee1fa9b88ac00000000
```

* （おまけ）署名後のトランザクションのRAWデータをデコードしてみる。
```
$ bitcoin-cli -testnet decoderawtransaction $SIGNED_RAW_TX
{
  "txid": "85d8de934e61052d611f5c97ea3218bd008bc53bc0d9b15b663779d661c74fe7",
  "hash": "85d8de934e61052d611f5c97ea3218bd008bc53bc0d9b15b663779d661c74fe7",
  "version": 2,
  "size": 341,
  "vsize": 341,
  "locktime": 0,
  "vin": [
    {
      "txid": "6d290b9beb5c0e18e13ba2939468416a2d2ac8c823673c094bfbffa2f0268c51",
      "vout": 1,
      "scriptSig": {
        "asm": "0 304502210093427236935cddad3afbe5a13ed583ac31e8407ca694fb7a8dc3c2785b91c70b022061a1c0c91ec46e53b726c5d54fd1b40e92c0152d8c676be7e606eff7dbf81423[ALL] 3045022100898ef207e8a56d415bb3e4a5e9b7c3033dc58c06947efd655eeba48a29ed7632022044808c589a2905fd3edaebf3d844e8ef7894aa833aea314f5248838982657d55[ALL] 52210314d9fbcd2d1afa935eecfe9faa3911b55c3d0dd3e0253dd5431f397a4a67e2d32102c17be5afd7837b6aad634b59113b646785b323117c562bf7ba70bf988164bf68210346c274aca653ba608b4163cd41b6b66f584d3e2e772bb349c18f9e6aff78eaba53ae",
        "hex": "0048304502210093427236935cddad3afbe5a13ed583ac31e8407ca694fb7a8dc3c2785b91c70b022061a1c0c91ec46e53b726c5d54fd1b40e92c0152d8c676be7e606eff7dbf8142301483045022100898ef207e8a56d415bb3e4a5e9b7c3033dc58c06947efd655eeba48a29ed7632022044808c589a2905fd3edaebf3d844e8ef7894aa833aea314f5248838982657d55014c6952210314d9fbcd2d1afa935eecfe9faa3911b55c3d0dd3e0253dd5431f397a4a67e2d32102c17be5afd7837b6aad634b59113b646785b323117c562bf7ba70bf988164bf68210346c274aca653ba608b4163cd41b6b66f584d3e2e772bb349c18f9e6aff78eaba53ae"
      },
      "sequence": 4294967295
    }
  ],
  "vout": [
    {
      "value": 0.03,
      "n": 0,
      "scriptPubKey": {
        "asm": "OP_DUP OP_HASH160 6049cdf6e6fc0151adad5f7200ebd3f61ee1fa9b OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "76a9146049cdf6e6fc0151adad5f7200ebd3f61ee1fa9b88ac",
        "reqSigs": 1,
        "type": "pubkeyhash",
        "addresses": [
          "mpJ5aVwC1BmVgHMVccsHdGTJNmh6u4XB1K"
        ]
      }
    }
  ],
  "hex": "0200000001518c26f0a2fffb4b093c6723c8c82a2d6a41689493a23be1180e5ceb9b0b296d01000000fdfe000048304502210093427236935cddad3afbe5a13ed583ac31e8407ca694fb7a8dc3c2785b91c70b022061a1c0c91ec46e53b726c5d54fd1b40e92c0152d8c676be7e606eff7dbf8142301483045022100898ef207e8a56d415bb3e4a5e9b7c3033dc58c06947efd655eeba48a29ed7632022044808c589a2905fd3edaebf3d844e8ef7894aa833aea314f5248838982657d55014c6952210314d9fbcd2d1afa935eecfe9faa3911b55c3d0dd3e0253dd5431f397a4a67e2d32102c17be5afd7837b6aad634b59113b646785b323117c562bf7ba70bf988164bf68210346c274aca653ba608b4163cd41b6b66f584d3e2e772bb349c18f9e6aff78eaba53aeffffffff01c0c62d00000000001976a9146049cdf6e6fc0151adad5f7200ebd3f61ee1fa9b88ac00000000"
}
```

* 署名済みのトランザクションをネットワークに送信する。
```
$ bitcoin-cli -testnet sendrawtransaction $SIGNED_RAW_TX
85d8de934e61052d611f5c97ea3218bd008bc53bc0d9b15b663779d661c74fe7
```

* 送信時のTXIDを変数に格納しておく。
```
$ export MULTISIG_SEND_TXID=85d8de934e61052d611f5c97ea3218bd008bc53bc0d9b15b663779d661c74fe7
$ echo $MULTISIG_SEND_TXID
85d8de934e61052d611f5c97ea3218bd008bc53bc0d9b15b663779d661c74fe7
```

* 送信直後のトランザクション情報は下記の通り確認できる。
* https://testnet.blockexplorer.com/tx/85d8de934e61052d611f5c97ea3218bd008bc53bc0d9b15b663779d661c74fe7
```
$ bitcoin-cli -testnet getrawtransaction $MULTISIG_SEND_TXID 1
{
  "hex": "0200000001518c26f0a2fffb4b093c6723c8c82a2d6a41689493a23be1180e5ceb9b0b296d01000000fdfe000048304502210093427236935cddad3afbe5a13ed583ac31e8407ca694fb7a8dc3c2785b91c70b022061a1c0c91ec46e53b726c5d54fd1b40e92c0152d8c676be7e606eff7dbf8142301483045022100898ef207e8a56d415bb3e4a5e9b7c3033dc58c06947efd655eeba48a29ed7632022044808c589a2905fd3edaebf3d844e8ef7894aa833aea314f5248838982657d55014c6952210314d9fbcd2d1afa935eecfe9faa3911b55c3d0dd3e0253dd5431f397a4a67e2d32102c17be5afd7837b6aad634b59113b646785b323117c562bf7ba70bf988164bf68210346c274aca653ba608b4163cd41b6b66f584d3e2e772bb349c18f9e6aff78eaba53aeffffffff01c0c62d00000000001976a9146049cdf6e6fc0151adad5f7200ebd3f61ee1fa9b88ac00000000",
  "txid": "85d8de934e61052d611f5c97ea3218bd008bc53bc0d9b15b663779d661c74fe7",
  "hash": "85d8de934e61052d611f5c97ea3218bd008bc53bc0d9b15b663779d661c74fe7",
  "version": 2,
  "size": 341,
  "vsize": 341,
  "locktime": 0,
  "vin": [
    {
      "txid": "6d290b9beb5c0e18e13ba2939468416a2d2ac8c823673c094bfbffa2f0268c51",
      "vout": 1,
      "scriptSig": {
        "asm": "0 304502210093427236935cddad3afbe5a13ed583ac31e8407ca694fb7a8dc3c2785b91c70b022061a1c0c91ec46e53b726c5d54fd1b40e92c0152d8c676be7e606eff7dbf81423[ALL] 3045022100898ef207e8a56d415bb3e4a5e9b7c3033dc58c06947efd655eeba48a29ed7632022044808c589a2905fd3edaebf3d844e8ef7894aa833aea314f5248838982657d55[ALL] 52210314d9fbcd2d1afa935eecfe9faa3911b55c3d0dd3e0253dd5431f397a4a67e2d32102c17be5afd7837b6aad634b59113b646785b323117c562bf7ba70bf988164bf68210346c274aca653ba608b4163cd41b6b66f584d3e2e772bb349c18f9e6aff78eaba53ae",
        "hex": "0048304502210093427236935cddad3afbe5a13ed583ac31e8407ca694fb7a8dc3c2785b91c70b022061a1c0c91ec46e53b726c5d54fd1b40e92c0152d8c676be7e606eff7dbf8142301483045022100898ef207e8a56d415bb3e4a5e9b7c3033dc58c06947efd655eeba48a29ed7632022044808c589a2905fd3edaebf3d844e8ef7894aa833aea314f5248838982657d55014c6952210314d9fbcd2d1afa935eecfe9faa3911b55c3d0dd3e0253dd5431f397a4a67e2d32102c17be5afd7837b6aad634b59113b646785b323117c562bf7ba70bf988164bf68210346c274aca653ba608b4163cd41b6b66f584d3e2e772bb349c18f9e6aff78eaba53ae"
      },
      "sequence": 4294967295
    }
  ],
  "vout": [
    {
      "value": 0.03,
      "n": 0,
      "scriptPubKey": {
        "asm": "OP_DUP OP_HASH160 6049cdf6e6fc0151adad5f7200ebd3f61ee1fa9b OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "76a9146049cdf6e6fc0151adad5f7200ebd3f61ee1fa9b88ac",
        "reqSigs": 1,
        "type": "pubkeyhash",
        "addresses": [
          "mpJ5aVwC1BmVgHMVccsHdGTJNmh6u4XB1K"
        ]
      }
    }
  ],
  "hex": "0200000001518c26f0a2fffb4b093c6723c8c82a2d6a41689493a23be1180e5ceb9b0b296d01000000fdfe000048304502210093427236935cddad3afbe5a13ed583ac31e8407ca694fb7a8dc3c2785b91c70b022061a1c0c91ec46e53b726c5d54fd1b40e92c0152d8c676be7e606eff7dbf8142301483045022100898ef207e8a56d415bb3e4a5e9b7c3033dc58c06947efd655eeba48a29ed7632022044808c589a2905fd3edaebf3d844e8ef7894aa833aea314f5248838982657d55014c6952210314d9fbcd2d1afa935eecfe9faa3911b55c3d0dd3e0253dd5431f397a4a67e2d32102c17be5afd7837b6aad634b59113b646785b323117c562bf7ba70bf988164bf68210346c274aca653ba608b4163cd41b6b66f584d3e2e772bb349c18f9e6aff78eaba53aeffffffff01c0c62d00000000001976a9146049cdf6e6fc0151adad5f7200ebd3f61ee1fa9b88ac00000000"
}
```

* トランザクション確定後
```
$ bitcoin-cli -testnet getrawtransaction $MULTISIG_SEND_TXID 1
{
  "hex": "0200000001518c26f0a2fffb4b093c6723c8c82a2d6a41689493a23be1180e5ceb9b0b296d01000000fdfe000048304502210093427236935cddad3afbe5a13ed583ac31e8407ca694fb7a8dc3c2785b91c70b022061a1c0c91ec46e53b726c5d54fd1b40e92c0152d8c676be7e606eff7dbf8142301483045022100898ef207e8a56d415bb3e4a5e9b7c3033dc58c06947efd655eeba48a29ed7632022044808c589a2905fd3edaebf3d844e8ef7894aa833aea314f5248838982657d55014c6952210314d9fbcd2d1afa935eecfe9faa3911b55c3d0dd3e0253dd5431f397a4a67e2d32102c17be5afd7837b6aad634b59113b646785b323117c562bf7ba70bf988164bf68210346c274aca653ba608b4163cd41b6b66f584d3e2e772bb349c18f9e6aff78eaba53aeffffffff01c0c62d00000000001976a9146049cdf6e6fc0151adad5f7200ebd3f61ee1fa9b88ac00000000",
  "txid": "85d8de934e61052d611f5c97ea3218bd008bc53bc0d9b15b663779d661c74fe7",
  "hash": "85d8de934e61052d611f5c97ea3218bd008bc53bc0d9b15b663779d661c74fe7",
  "version": 2,
  "size": 341,
  "vsize": 341,
  "locktime": 0,
  "vin": [
    {
      "txid": "6d290b9beb5c0e18e13ba2939468416a2d2ac8c823673c094bfbffa2f0268c51",
      "vout": 1,
      "scriptSig": {
        "asm": "0 304502210093427236935cddad3afbe5a13ed583ac31e8407ca694fb7a8dc3c2785b91c70b022061a1c0c91ec46e53b726c5d54fd1b40e92c0152d8c676be7e606eff7dbf81423[ALL] 3045022100898ef207e8a56d415bb3e4a5e9b7c3033dc58c06947efd655eeba48a29ed7632022044808c589a2905fd3edaebf3d844e8ef7894aa833aea314f5248838982657d55[ALL] 52210314d9fbcd2d1afa935eecfe9faa3911b55c3d0dd3e0253dd5431f397a4a67e2d32102c17be5afd7837b6aad634b59113b646785b323117c562bf7ba70bf988164bf68210346c274aca653ba608b4163cd41b6b66f584d3e2e772bb349c18f9e6aff78eaba53ae",
        "hex": "0048304502210093427236935cddad3afbe5a13ed583ac31e8407ca694fb7a8dc3c2785b91c70b022061a1c0c91ec46e53b726c5d54fd1b40e92c0152d8c676be7e606eff7dbf8142301483045022100898ef207e8a56d415bb3e4a5e9b7c3033dc58c06947efd655eeba48a29ed7632022044808c589a2905fd3edaebf3d844e8ef7894aa833aea314f5248838982657d55014c6952210314d9fbcd2d1afa935eecfe9faa3911b55c3d0dd3e0253dd5431f397a4a67e2d32102c17be5afd7837b6aad634b59113b646785b323117c562bf7ba70bf988164bf68210346c274aca653ba608b4163cd41b6b66f584d3e2e772bb349c18f9e6aff78eaba53ae"
      },
      "sequence": 4294967295
    }
  ],
  "vout": [
    {
      "value": 0.03,
      "n": 0,
      "scriptPubKey": {
        "asm": "OP_DUP OP_HASH160 6049cdf6e6fc0151adad5f7200ebd3f61ee1fa9b OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "76a9146049cdf6e6fc0151adad5f7200ebd3f61ee1fa9b88ac",
        "reqSigs": 1,
        "type": "pubkeyhash",
        "addresses": [
          "mpJ5aVwC1BmVgHMVccsHdGTJNmh6u4XB1K"
        ]
      }
    }
  ],
  "hex": "0200000001518c26f0a2fffb4b093c6723c8c82a2d6a41689493a23be1180e5ceb9b0b296d01000000fdfe000048304502210093427236935cddad3afbe5a13ed583ac31e8407ca694fb7a8dc3c2785b91c70b022061a1c0c91ec46e53b726c5d54fd1b40e92c0152d8c676be7e606eff7dbf8142301483045022100898ef207e8a56d415bb3e4a5e9b7c3033dc58c06947efd655eeba48a29ed7632022044808c589a2905fd3edaebf3d844e8ef7894aa833aea314f5248838982657d55014c6952210314d9fbcd2d1afa935eecfe9faa3911b55c3d0dd3e0253dd5431f397a4a67e2d32102c17be5afd7837b6aad634b59113b646785b323117c562bf7ba70bf988164bf68210346c274aca653ba608b4163cd41b6b66f584d3e2e772bb349c18f9e6aff78eaba53aeffffffff01c0c62d00000000001976a9146049cdf6e6fc0151adad5f7200ebd3f61ee1fa9b88ac00000000",
  "blockhash": "00000000a81f90f4dc38d7c3271fd8d53f2cd9e5ec713c26bfc30a61f17152e1",
  "confirmations": 1,
  "time": 1496391925,
  "blocktime": 1496391925
}
```

* トランザクション承認後、残高を確認してみる（``listunspent``）。アドレス４に残高が増えていることを確認できるはずである。
```
$ bitcoin-cli -testnet listunspent
[
  （中略）
  {
    "txid": "85d8de934e61052d611f5c97ea3218bd008bc53bc0d9b15b663779d661c74fe7",
    "vout": 0,
    "address": "mpJ5aVwC1BmVgHMVccsHdGTJNmh6u4XB1K",
    "account": "",
    "scriptPubKey": "76a9146049cdf6e6fc0151adad5f7200ebd3f61ee1fa9b88ac",
    "amount": 0.03000000,
    "confirmations": 452,
    "spendable": true,
    "solvable": true,
    "safe": true
  }
]
```

## 1-4-3. Bitcoin における MultiSig について
* 署名の断面の状態をネットワークで保存できない（ステートレス）。
  * Bitcoin（およびCounterparty等のBitcoin上で動くカラードコインも同様）のMultiSigでは、上記の例のように、アドレス１が署名→アドレス３が署名→トランザクション送信→トランザクション承認という流れになる。
  * アドレス１が署名した後の中間状態で、ネットワークはそのトランザクションを承認できない。
  * アドレス１が署名した後のhexをセキュアな領域に保存し、それをアドレス３に追加署名してもらう必要がある。
  * その他 Redeem Script など、保存・共有する必要がある情報がある点に注意が必要である。
* 一方で、EthereumのMultiSigは署名の断面の状態をネットワークで保存することができる。
  * BitcoinのMultiSigとは異なり、スマートコントラクトで実装を行う。スマートコントラクトのデータはステートフルであるため、EthereumのMultiSigはステートフルに扱える。
  * Ethereumにおいては、”コントラクトアカウント”がMultiSigアドレスの役割を担い、Redeem ScriptなどのUTXOモデル独自の概念もない。
  * https://dappsforbeginners.wordpress.com/tutorials/two-party-contracts/
* 一見同じに思えるMultiSigもプラットフォームによって実装の方法が異なってくる。
