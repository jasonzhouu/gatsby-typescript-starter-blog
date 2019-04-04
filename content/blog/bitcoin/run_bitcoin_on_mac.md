---
title: 【Bitcoin Core系列】在Mac上运行
date: '2018-12-29'
---

安装包下载链接：https://bitcoincore.org/en/download/，Mac 64位的14.1MB。
这个安装包安装的是bitcoin-qt，官网不提供bitcoind的安装包，如果需要的话可以自己编译。[^1]

## mainnet, testnet, regtest 模式
Bitcoin有这3种模式，默认用的是mainnet模式。打开Bitcoin客户端后会提示需要同步200多GB的block，可以更改保存路径，比如将数据保存在移动硬盘上。我之前在Ubuntu上试过同步mainnet的完整区块链，用了大约1天时间，晚上会特别快。

但是如果只是想熟悉Bitcoin的命令或者基于Bitcoin开发，可以不用mainnet模式，改用 testnet或者regtest 模式，灵活度更大[^5]，测试时也不需要买币。
- testnet是模拟版的mainnet，同样需要同步整条链。里面的币是没有价值的，可以[向别人要一些](https://tpfaucet.appspot.com/)用来进行测试。[^4]
- regtest是private chain，它只运行在你的本地，不和其他peer交互。灵活度最大，刚打开的时候区块数据是空的，可以自行创建。[^3]

## 运行testnet模式
### 方法1：修改配置文件
修改`~/Library/Application Support/Bitcoin/bitcoin.conf`文件，添加：[^6]
```
testnet=1
```

另外一种打开配置文件的方式：
bitcoin-qt  ->  preferences  ->  Open Configuration File
 
### 方法2：命令行中指定参数
bitcoin-qt命令位于 `/Applications/Bitcoin-Qt.app/Contents/MacOS/Bitcoin-Qt`

运行命令：
`/Applications/Bitcoin-Qt.app/Contents/MacOS/Bitcoin-Qt -testnet`

testnet的数据保存在`/Users/yushengzhou/Library/Application Support/Bitcoin/testnet3`

如果在软件界面顶部看到 `[testnet]`，说明设置成功。

![](https://upload-images.jianshu.io/upload_images/3963416-6c450505ccfa6a64.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## console
和Linux版本的bitcoin-qt不同的是，Mac版本没有`bitcoin-cli`命令行工具[^2]，只能在Bitcoin图形界面的console执行命令。打开步骤是：help->debug window->console。使用体验其实挺不错的，输入命令时会有提示，可以减少记忆负担。

![](https://upload-images.jianshu.io/upload_images/3963416-d07b6965737a0c71.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



[^1]: https://bitcoin.stackexchange.com/a/61548/81787
[^2]: https://bitcoin.stackexchange.com/a/43160/81787
[^3]: https://bitcoin.org/en/glossary/regression-test-mode
[^4]: https://bitcoin.org/en/glossary/testnet
[^5]: https://bitcoin.org/en/developer-examples#testing-applications
[^6]: https://jeiwan.cc/posts/what-is-lightning-network/
