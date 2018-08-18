## 智能合约入门

#### 一、什么是智能合约？

智能合约是一个全网协同计算机上的小程序，用来计算和存储数据。

智能合约通过外部调用其函数以及变量，来与客户端进行数据交互。

#### 二、一个最简单的智能合约

```
pragma solidity ^0.4.11;

contract Setdata {
    string public ipfsHash;
    uint256 public baseSquare;
 	
 	constructor() public {
 		//构造函数
 	}
 	
    function setHash(string x) public {
        ipfsHash = x;
    }

    function getHash() public constant returns (string x) {
        return ipfsHash;
    }
    
    function setBase(uint256 a) public {
    	baseSquare = a * a;
    }
}
```

#### 如何调试智能合约？

介绍五种方法，以太坊开发生态已经很成熟，调试与部署远远不止这些方案。
##### 1- 部署在本地`Ganache`环境中

##### 2- 在本地搭建以太坊环境，部署并调试（安装太麻烦，不提倡）

##### 3- 在Remix中调试，并部署在测试网络中，调试成功后，一键部署到主网上。

如：`Ropsten`或者`Rinkeby`网络。

优点：调试方便，所见即所得；

##### 4- 在`truffle`的`develop`环境中调试

运行`truffle develop`后，会在本地启动一个以太坊微环境以及10个帐号，供调试。

##### 5- 使用`infura`的API，通过`Metamask`的`HDWalletProvider`通道部署到测试网络中

建议较大的智能合约，而且无法从`github`上直接导入库的时候，使用此方法。

（以上方法将在下节课中讲述）
