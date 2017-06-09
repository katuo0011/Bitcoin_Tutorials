# 1-3. Bitcoinのトランザクション

-----------------------------------------------------------
## 1-3-1. トランザクションデータ
* サンプルとして単純な送金（P2PKH）のトランザクションの内容を以下に再掲する。※1-1-6でサンプル送金したトランザクションの中身。
* Bitcoinのトランザクションは、主にインプットトランザクションに対する署名情報(``vin``）、アウトプットトランザクションの情報（``vout``）で構成される。

```
$ bitcoin-cli -testnet getrawtransaction 7b771a44f57be79b16983787ef9634473b850b662ea4c5b3d76af96984cf28e2 1
{
  "hex": "0200000001b99199f8631ede2b62c51c071396e73192856f9aa324f8f8059295dfb4e90a3c010000006b483045022100c2fecae10ee4eeff1da732e4eeaf6a639af00cd6e0be5e560d4c5f6c02f5785b022020278b0cc448c932fccaf24d6122b7f1c451c7c22039d0efd3a12589a86faf5c012103714f5e2e1b7905b11d065c107bea605b936d64762feb6b48924ea3567d0d6c4efeffffff02a0860100000000001976a914b70bc96364e73106bf1f51aac5aebc1eae0190a888ac11070e07000000001976a914ad963390a71e668d44ad19f0fc83d4e1e6699b6588acbb2b1100",
  "txid": "7b771a44f57be79b16983787ef9634473b850b662ea4c5b3d76af96984cf28e2",
  "hash": "7b771a44f57be79b16983787ef9634473b850b662ea4c5b3d76af96984cf28e2",
  "version": 2,
  "size": 226,
  "vsize": 226,
  "locktime": 1125307,
  "vin": [
    {
      "txid": "3c0ae9b4df959205f8f824a39a6f859231e79613071cc5622bde1e63f89991b9",
      "vout": 1,
      "scriptSig": {
        "asm": "3045022100c2fecae10ee4eeff1da732e4eeaf6a639af00cd6e0be5e560d4c5f6c02f5785b022020278b0cc448c932fccaf24d6122b7f1c451c7c22039d0efd3a12589a86faf5c[ALL] 03714f5e2e1b7905b11d065c107bea605b936d64762feb6b48924ea3567d0d6c4e",
        "hex": "483045022100c2fecae10ee4eeff1da732e4eeaf6a639af00cd6e0be5e560d4c5f6c02f5785b022020278b0cc448c932fccaf24d6122b7f1c451c7c22039d0efd3a12589a86faf5c012103714f5e2e1b7905b11d065c107bea605b936d64762feb6b48924ea3567d0d6c4e"
      },
      "sequence": 4294967294
    }
  ],
  "vout": [
    {
      "value": 0.001,
      "n": 0,
      "scriptPubKey": {
        "asm": "OP_DUP OP_HASH160 b70bc96364e73106bf1f51aac5aebc1eae0190a8 OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "76a914b70bc96364e73106bf1f51aac5aebc1eae0190a888ac",
        "reqSigs": 1,
        "type": "pubkeyhash",
        "addresses": [
          "mxCp57QvezWaMoLGmCPFxgcohtbAE1Fmbb"
        ]
      }
    },
    {
      "value": 1.18359825,
      "n": 1,
      "scriptPubKey": {
        "asm": "OP_DUP OP_HASH160 ad963390a71e668d44ad19f0fc83d4e1e6699b65 OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "76a914ad963390a71e668d44ad19f0fc83d4e1e6699b6588ac",
        "reqSigs": 1,
        "type": "pubkeyhash",
        "addresses": [
          "mwLo8REpktdLhjVCLzmcTmFUNxXkxbCriW"
        ]
      }
    }
  ],
  "hex": "0200000001b99199f8631ede2b62c51c071396e73192856f9aa324f8f8059295dfb4e90a3c010000006b483045022100c2fecae10ee4eeff1da732e4eeaf6a639af00cd6e0be5e560d4c5f6c02f5785b022020278b0cc448c932fccaf24d6122b7f1c451c7c22039d0efd3a12589a86faf5c012103714f5e2e1b7905b11d065c107bea605b936d64762feb6b48924ea3567d0d6c4efeffffff02a0860100000000001976a914b70bc96364e73106bf1f51aac5aebc1eae0190a888ac11070e07000000001976a914ad963390a71e668d44ad19f0fc83d4e1e6699b6588acbb2b1100",
  "blockhash": "000000000000020de3243b300319122d77c76a080b7f0bf6997736527d6bcd67",
  "confirmations": 1833,
  "time": 1495439916,
  "blocktime": 1495439916
}
```
-----------------------------------------------------------
## 1-3-2. output(vout)：scriptPubKey, locking script
* ``vout`` にはscriptPubKeyと呼ばれるスクリプト処理が記述される。
* 単純な送金（P2PKH）の場合、``OP_DUP OP_HASH160 （公開鍵のハッシュ値） OP_EQUALVERIFY OP_CHECKSIG``で記述される。※送信先アドレスから公開鍵そのものを直接生成することはできない。ビットコインアドレスは公開鍵のハッシュ値を人間の目でも判別できる文字列に変換したもの。
* scriptPubKey は locking script とも呼ばれる。
* 上記スクリプトは、「次にトランザクションを生成する際に、scriptSig（後述）内に対応する署名が含まれていなければならない」という意味である。
* このスクリプトを満たさなければ、トランザクションは生成できない。つまり、秘密鍵の所有者以外ビットコインを使用できないということになる。

```
"vout": [
  {
    "value": 0.001,
    "n": 0,
    "scriptPubKey": {
      "asm": "OP_DUP OP_HASH160 b70bc96364e73106bf1f51aac5aebc1eae0190a8 OP_EQUALVERIFY OP_CHECKSIG",
      "hex": "76a914b70bc96364e73106bf1f51aac5aebc1eae0190a888ac",
      "reqSigs": 1,
      "type": "pubkeyhash",
      "addresses": [
        "mxCp57QvezWaMoLGmCPFxgcohtbAE1Fmbb"
      ]
    }
  },
  {
    "value": 1.18359825,
    "n": 1,
    "scriptPubKey": {
      "asm": "OP_DUP OP_HASH160 ad963390a71e668d44ad19f0fc83d4e1e6699b65 OP_EQUALVERIFY OP_CHECKSIG",
      "hex": "76a914ad963390a71e668d44ad19f0fc83d4e1e6699b6588ac",
      "reqSigs": 1,
      "type": "pubkeyhash",
      "addresses": [
        "mwLo8REpktdLhjVCLzmcTmFUNxXkxbCriW"
      ]
    }
  }
],
```

-----------------------------------------------------------
## 1-3-3. input(vin)：scriptSig, unlocking script
* ``vin``には、もとの ``txid``と``vout`` (UTXO: unspent TX output) ``scriptSig``が定義されている。
* scriptSig はUTXOの所有者による署名データである。ネットワークでは、この署名情報がUTXOの所有者が行った署名であることを検証する（公開鍵暗号方式）。
* scriptSigは ``vout`` のlockingを解除するという意味で unlocking script とも呼ばれる。

```
"vin": [
  {
    "txid": "3c0ae9b4df959205f8f824a39a6f859231e79613071cc5622bde1e63f89991b9",
    "vout": 1,
    "scriptSig": {
      "asm": "3045022100c2fecae10ee4eeff1da732e4eeaf6a639af00cd6e0be5e560d4c5f6c02f5785b022020278b0cc448c932fccaf24d6122b7f1c451c7c22039d0efd3a12589a86faf5c[ALL] 03714f5e2e1b7905b11d065c107bea605b936d64762feb6b48924ea3567d0d6c4e",
      "hex": "483045022100c2fecae10ee4eeff1da732e4eeaf6a639af00cd6e0be5e560d4c5f6c02f5785b022020278b0cc448c932fccaf24d6122b7f1c451c7c22039d0efd3a12589a86faf5c012103714f5e2e1b7905b11d065c107bea605b936d64762feb6b48924ea3567d0d6c4e"
    },
    "sequence": 4294967294
  }
]
```
