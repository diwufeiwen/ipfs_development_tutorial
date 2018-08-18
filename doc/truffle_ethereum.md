## Truffle以太坊开发环境搭建
### By Jacky古千峰

#### 一、安装`Truffle`
```
npm install -g truffle
```

#### 二、创建工程
##### 1- 创建一个空工程
```
truffle init
```

##### 2- 创建包含metacoin的工程
新版本truffle引入了box的概念，所有的示例代码都以box的形式提供。下载metacoin的示例代码：

```
truffle unbox metacoin
```
[点击这里，查看truffle官方提供的Box](https://truffleframework.com/boxes)，可以方便的使用`truffle unbox`将代码框架下载到本地使用。

#### 三、安装以太坊客户端Ganache
智能合约必须要部署到链上进行测试。可以选择部署到一些公共的测试链比如 Rinkeby 或者 Ropsten 上，缺点是：部署和测试时间比较长，需要申请一些假的代币。所以对于开发者，最好的方式是部署到私链上。

Ganache是​​以太坊开发的个人区块链。他的前身是 testRPC ，很多旧的教程介绍的都是 testRPC 。

##### 1- 图形界面Ganache的安装方式，[点击这里，根据不同的系统版本下载](https://github.com/trufflesuite/ganache/releases)。

直接安装即可。

##### 2- 命令行界面安装
```
npm install -g ganache-cli  
```

#### 四、配置truffle.js
Ganache 默认运行在本地 7545 端口，运行后默认创建10个账号，每个账号里有100ETH的余额。

![](http://images.laidingyi.com/18-8-7/24473863.jpg)

打开生成的`metacoin`目录下的truffle.js，将以下配置文件粘贴到文件中：

```
module.exports = {
  networks: {
    development: {
      host: 'localhost',
      port: '7545',
      network_id: '*' // Match any network id
    }
  }
};
```

#### 五、测试环境
##### 编译并部署Metacoin合约
```
truffle compile  
truffle migrate
```

然后，测试该合约
```
truffle test
```

如果出现以下结果，说明测试通过。

![](http://images.laidingyi.com/18-8-7/97919844.jpg)

#### 六、以太坊智能合约调试与部署方案小结

##### 1- 部署在本地`Ganache`环境中

##### 2- 在本地搭建以太坊环境，部署并调试（安装太麻烦，不提倡）

##### 3- 在Remix中调试，并部署在测试网络中，调试成功后，一键部署到主网上。

如：`Ropsten`或者`Rinkeby`网络。

优点：调试方便，所见即所得；

缺点：有些从`github`上`import`的库无法成功调试，如`zepellin-solidity`

##### 4- 在`truffle`的`develop`环境中调试

运行`truffle develop`后，会在本地启动一个以太坊微环境以及10个帐号，供调试。

##### 5- 使用`infura`的API，通过`Metamask`的`HDWalletProvider`通道部署到测试网络中

建议较大的智能合约，而且无法从`github`上直接导入库的时候，使用此方法。

（以上方法将在下节课中讲述）
