---
title: 【Bitcoin】script
date: '2019-01-02'
---


Bitcoin使用的是UTXO模式，一笔交易得包含至少一个vin和一个vout，还没有被使用的vout称为UTXO(unspent transaction output)，只要你能够解锁UTXO中的scriptPubKey，你就能消费其中的余额。

## script
scriptPubKey不是简单的就是一个公钥，而是一段script代码，它决定了消费这笔钱的条件。它由bitcoin规定的script words组成。[^1]前面提到的“解锁scriptPubKey”指的是“你能给scriptPubKey传入数值，使它返回的结果为true”。

不同的script可以支持不同的应用场景，比如常用的P2PKH、P2SH交易，前者用于支付给个人，后者使用多重签名，需要多个人同时签名才能消费。他们的scriptPubKey分别是这样的：

- P2PKH: `OP_DUP OP_HASH160 <pubKeyHash> OP_EQUALVERIFY OP_CHECKSIG`
- P2SH: `OP_HASH160 <scriptHash> OP_EQUAL`

一点代码的改变，就可以创建完全不同的逻辑，决定需要满足什么条件才能消费这笔钱。这也是bitcoin script的强大之处。通过编写script，甚至可以支持侧链、闪电网络。

## P2PKH交易

`P2PKH`(pay to pulic key hash)是最常见的交易形式，用于支付给个人，它的script的形式是这样的：

```
scriptPubKey: OP_DUP OP_HASH160 <pubKeyHash> OP_EQUALVERIFY OP_CHECKSIG
scriptSig: <sig> <pubKey>
```

它用到的命令`OP_DUP`, `OP_HASH160`, `OP_EQUALVERIFY`, `OP_CHECKSIG`在 https://en.bitcoin.it/wiki/Script 的对照表中都可以找到对应的意义。比如`OP_HASH160`表示执行两次hash运算，依次用SHA-256、RIPEMD-160算法。

![opcode对照表](https://upload-images.jianshu.io/upload_images/3963416-00414d4e0607a35b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 地址
scriptPubKey的`<pubKeyHash>`可以看作是P2PKH交易的地址，也就是说接收转账的时候需要告诉对方的是自己的`<pubKeyHash>`(等于`RIPEMD-160(SHA-256(public key))`)，而不是public key本身。

为了迎合人的习惯，减少地址的长度，可以对地址进行Base58编码。相比Base64编码，Base58编码不使用(0, O, l, I)，因为人容易搞混淆。然后还会在地址前面加上地址使用的规范的版本号，以实现向后兼容。编码后的地址是给人看的，计算机处理的时候会先解码。

生成地址的过程[^4]：
![](https://upload-images.jianshu.io/upload_images/3963416-a17a513704c4bf88.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 举例
比如有一笔交易，Alice转出500个BTC给Bob。使用命令`bitcoin-cli getrawtransaction txid 1`查看交易的完整信息，它包含了交易的scriptSig, scriptPubKey。（末尾加上1，表示将返回可读形式的结果，否则得到是hex格式。）

```
$ getrawtransaction 963e9c337a8d7277194313abd2df70646e5cc1c8d7e36ebf2acf5d8fd66e10dc 1
```

返回结果包括vin和vout
### vin
vin需要指定它UTXO的txid，和用于解锁这个UTXO的scriptSig。
```
"vin": [
    {
      "txid": "75ff1170e017ba20bb20a9bf2f3c7cd4390a72f1d5eb4af35cbd8851aa303fb6",
      "vout": 1,
      "scriptSig": {
        "asm": "0014bd392459da24fbbd84beb55604f2e9a0d9524b1e",
        "hex": "160014bd392459da24fbbd84beb55604f2e9a0d9524b1e"
      },
      "txinwitness": [
        "3044022003b07eafbddef7e8d36b1591b39166fc01810e0490d7fbe88cb77e5cc38cf75c0220015db647488ecb83a1ab467880698ac7f32beb74c3ae3c1bab14fc0e9fade68601",
        "022058a9a3a8e1415b7db0aca473c1df5d47634a0da44e4c25dc2a14739d189ee4"
      ],
      "sequence": 4294967293
    }
  ],
```
### vout
vout需要指定转账金额，和这些金额的解锁条件scriptPubKey。
```
 "vout": [
    {
      "value": 500.00000000,
      "n": 0,
      "scriptPubKey": {
        "asm": "OP_DUP OP_HASH160 e7c1345fc8f87c68170b3aa798a956c2fe6a9eff OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "76a914e7c1345fc8f87c68170b3aa798a956c2fe6a9eff88ac",
        "reqSigs": 1,
        "type": "pubkeyhash",
        "addresses": [
          "n2eMqTT929pb1RDNuqEnxdaLau1rxy3efi"
        ]
      }
    },
    ...
  ],
```
Alice转账500BTC给Bob后，Bobå建了新的交易，转了一些BTC给Carol。Bob的这笔转账交易的vin需要包含scriptSig，P2PKH交易的scriptSig的格式是`<sig> <pubKey>`，即公钥加电子签名。

矿工收到交易后，会对其进行验证。验证内容是：前一笔交易（Alice转账给Bob）的vout中scriptPubKey，和新交易（Bob转账给Carol）的vin中的scriptSig。

- 先将scriptSig添加到scriptPubKey前面，拼起来，得到`<Sig> <PubKey> OP_DUP OP_HASH160 <PubkeyHash> OP_EQUALVERIFY OP_CHECKSIG`。
- 计算前一步得到的script，如果结果为true，则通过验证。[^2]

### script的计算

`<Sig> <PubKey> OP_DUP OP_HASH160 <PubkeyHash> OP_EQUALVERIFY OP_CHECKSIG`

script从左到右一步一步，基于栈模式执行：

![](https://upload-images.jianshu.io/upload_images/3963416-0fa5cfa64b7982ea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


计算步骤：[^2][^3]
1. `<Sig>`: 存入<Sig>
2. `<PubKey>`: 存入<PubKey>
3. `OP_DUP`：复制一份<PubKey>添加到栈（因为最后一步OP_CHECKSIG要用到）`
4. `OP_HASH160`：计算<PubKey>的hash
5. `<PubkeyHash>`：存入<PubkeyHash>
6. `OP_EQUALVERIFY`：比较前面两个hash是否相等，如果相等就继续执行，否则退出
7. `OP_CHECKSIG`：使用<PubKey>检验<Sig>，只有由<PubKey>对应的私钥创建的<Sig>才能通过检验。因为只有Bob有这个私钥，所以只有他创建的<Sig>才能通过检验，从而也就只有它能消费这个UTXO。

## todo

[x] <Sig>是对什么内容的签名？有多种情况，详情见 [https://bitcoin.org/en/developer-guide#signature-hash-types](https://bitcoin.org/en/developer-guide#signature-hash-types)

[x] P2PKH相比P2PK有什么优势吗，为什么要多加个hash环节？缩短交易地址，提高安全性
[x] P2SH(pay to script hash)的地址是script的hash，具体是怎么实现multisig的？mutlsig 可以脱离P2SH单独实现，原理和P2PK差不多，只是需要多验证几个签名。基于P2SH实现的multisig更加灵活。详情见"Bitcoin Development Reference"的4.3.4 Multisig部分。

## 参考

[^1]: https://en.bitcoin.it/wiki/Script
[^2]: https://bitcoin.org/en/developer-guide#p2pkh-script-validation
[^3]: https://en.bitcoin.it/wiki/Transaction#Pay-to-PubkeyHash
[^4]: "Mastering Bitcoin" 的 Bitcoin Address 章节
