## 基于zeppelin创建一个安全简单的ERC20合约
#### By Jacky 古千峰@BTCMedia、IPFSForce

本教程来介绍下`ERC20`合约，以及如何使用`zeppelin`合约开发组件来快速、安全的发行`ERC20`合约，即代币。

之所以用`zeppelin`开发ERC20合约，很简单，因为`zeppelin`是目前为止最安全的开发组件，所有代码经严格审计。到目前为止，基于该框架开发的智能合约，尚未发现漏洞。前些时间，国内发生了多起智能合约安全事件，都无一使用`zeppelin`框架。

建议没事经常看看`zeppelin`的智能合约。

#### 什么是ERC20合约

ERC20合约也成为代币合约，由以下6个标准函数组成：

|函数|方法|
|---|---|
|totalSupply()|代币发行的总量|
|balanceOf(A)|查询A帐户下的代币数目|
|transfer(A,x)|发送x个代币到A帐户|
|transferFrom(A,x)|从A帐户提取x个代币|
|approve(A,x)|同意A帐户从我的帐户中提取代币|
|allowance(A,B)|查询B帐户可以从A帐户提取多少代币|

#### 如何使用zeppelin开发ERC20合约

#### 1- 创建`Truffle`项目框架

新建项目目录，进到该目录，输入：

```
truffle init
npm init
npm install zeppelin-solidity
```

#### 2- 创建合约

进到`contracts`目录，创建文件`ERC20_A.sol`，在该文件中输入：

```
pragma solidity ^0.4.16;

import "zeppelin-solidity/contracts/token/ERC20/StandardToken.sol";

contract ERC20_A is StandardToken {
    address public owner;                     // 所有人
    string public name = "EOSHACKATION";      // 代币名称
    string public symbol = "EOSIO";           // 代币符号
    uint8 public decimals = 18;               // 代币精度
    uint256 public INITIAL_SUPPLY = 1000000000000000000000000000; // 总量10亿

    constructor() public {
        totalSupply_ = INITIAL_SUPPLY;
        balances[msg.sender] = INITIAL_SUPPLY;
        owner = msg.sender;
    }
}
```

以上就是基于`zeppelin`的`ERC20`合约代码。

#### 3- 部署合约

进到`migrations`目录，创建文件`2_deploy_migration.js`文件，输入：

```
var Eoshackathon = artifacts.require("./ERC20_A.sol");

module.exports = function (deployer) {
    deployer.deploy(Eoshackathon);
};
```

#### 4- 在truffle中调试合约

* 执行命令：

```
truffle develop
```
显示如下内容：

```
Truffle Develop started at http://127.0.0.1:9545/

Accounts:
(0) 0x627306090abab3a6e1400e9345bc60c78a8bef57
(1) 0xf17f52151ebef6c7334fad080c5704d77216b732
(2) 0xc5fdf4076b8f3a5357c5e395ab970b5b54098fef
(3) 0x821aea9a577a9b44299b9c15c88cf3087f3b5544
(4) 0x0d1d4e623d10f9fba5db95830f7d3839406c6af2
(5) 0x2932b7a2355d6fecc4b5c0b6bd44cc31df247a2e
(6) 0x2191ef87e392377ec08e7c08eb105ef5448eced5
(7) 0x0f4f2ac550a1b4e2280d04c21cea7ebd822934b5
(8) 0x6330a553fc93768f612722bb8c2ec78ac90b3bbc
(9) 0x5aeda56215b167893e80b4fe645ba6d5bab767de

Private Keys:
(0) c87509a1c067bbde78beb793e6fa76530b6382a4c0241e5e4a9ec0a0f44dc0d3
(1) ae6ae8e5ccbfb04590405997ee2d52d2b330726137b875053c36d94e974d162f
(2) 0dbbe8e4ae425a6d2687f1a7e3ba17bc98c673636790f1b8ad91193c05875ef1
(3) c88b703fb08cbea894b6aeff5a544fb92e78a18e19814cd85da83b71f772aa6c
(4) 388c684f0ba1ef5017716adb5d21a053ea8e90277d0868337519f97bede61418
(5) 659cbb0e2411a44db63778987b1e22153c086a95eb6b18bdf89de078917abc63
(6) 82d052c865f5763aad42add438569276c00d3d88a2d062d36b2bae914d58b8c8
(7) aa3680d5d48a8283413f7a108367c7299ca73f553735860a87b08f39395618b7
(8) 0f62d96d6675f32685bbdb8ac13cda7c23436f63efbb9d07700d8669ff12b7c4
(9) 8d5366123cb560bb606379f90a0bfd4769eecc0557f1b362dcae9012b548b1e5
```
测试程序自动生成了10个测试帐号。

* 执行命令：`compile` 编译合约

* 执行命令：`migrate` 部署合约，合约部署完后，显示如下信息：

```
Running migration: 2_deploy_migration.js
  Deploying ERC20_A...
  ... 0x9354ef38544ee9eab548bd5b5830763116075941a3f5ab9539680bd7b701d4f3
  ERC20_A: 0x345ca3e014aaf5dca488057592ee47305d9b3e10    //合约地址
Saving successful migration to network...                
  ... 0xf36163615f41ef7ed8f4a8f192149a0bf633fe1a2398ce001bf44c43dc7bdda0
Saving artifacts...
```
* 验证是否已经部署，执行：

```
ERC20_A.deployed().then(instance => contract = instance)
```
此命令将ERC20_A合约的实例赋值给`contract`变量。

* 查看`coinbase`代币发行总数，执行以下命令。`balanceOf`是6个标准的ERC20函数之一。

```
contract.balanceOf(web3.eth.coinbase)
```

* 向账户#1打币，

```
contract.transfer(web3.eth.accounts[1], 600000)
```

* 查看#1账户余额
```
contract.balanceOf(web3.eth.accounts[1])
```

*在develop模式下，可以调试所有的web3.js命令*

#### 5- 在Remix中调试合约
将`import`语句改为如下即可：

```
import "github.com/OpenZeppelin/zeppelin-solidity/contracts/token/ERC20/StandardToken.sol";
```
在编译时，会自动将`github`的库导入。

调试时，在`Remix`工具中的合约地址中，填入部署的合约地址即可。


#### 6- 将合约部署到测试网络Ropsten

首先到[infura.io](https://infura.io/)，注册账户，并获取`API key`。

然后到`Metamask`的`setting`中的`Reveal Seed Words`，复制12个字符助记词。

安装`truffle-hdwallet-provider`模块。

```
npm install truffle-hdwallet-provider --save
```

然后，修改`ruffle.js`，将以下代码贴入：

```
var HDWalletProvider = require("truffle-hdwallet-provider");

var infura_apikey = "api_key";
var mnemonic = "12个助记词";

module.exports = {
    networks: {
        development: {
            host: "127.0.0.1",
            port: 8545,
            network_id: "*" // Match any network id
        },
        ropsten: {
            provider: new HDWalletProvider(mnemonic, "https://ropsten.infura.io/" + infura_apikey),
            network_id: 3
        }
    }
};
```

最后，执行以下命令，部署合约：

```
truffle migrate --network ropsten
```
在浏览器中打开生成合约的合约地址，查看该合约部署情况。

