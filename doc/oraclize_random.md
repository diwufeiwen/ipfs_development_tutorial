## 使用预言机Oracle获取真实随机数以及链下数据
### By 古千峰 Jacky@BTCMedia、IPFSForce

在区块链世界，最大的问题是区块链链上数据如何与现实世界的数据进行连接。

其中最常遇到的就是随机数，在很多游戏应用中，都需要使用随机数。很多人使用未来某一时间点的区块链高度或者hash值生成，但这种方式很容易被矿工"猜到"，非常不安全。

到目前为止，安全的随机数只有通过Oracle（预言机）方式获得。

本教程介绍如何通过`Oraclize`获得真实的随机数。

### 一、Oraclize介绍
Oraclize是目前使用最多的收费预言机服务，可以获取将任意网站的数据、IPFS数据、WolframAlpha以及随机数转到链上进行处理。

价格表参考[链接](https://docs.oraclize.it/#pricing-advanced-datasources-call-fee)

每种类型的数据有不同的基础价格，另外，如果需要采取验证服务`TLSNotary`（也就是验证整个数据传输过程没有被篡改），还需要另外加上额外费用。

### 二、获取随机数
以随机数为例，每获取一次随机数，需要支付`$0.05`，推荐另外一种方式：以`URL`从`random.org`上获取随机数。

为什么要从`random.org`上获取呢？因为`random.org`是一个权威的随机数生成工具，如果自己用php写一个随机数程序，放在服务器上，让智能合约去调用，别人不一定相信。

另外，还有一个原因是`random.org`可以有多种方式获取随机数，包括一次性生成多个随机数，这些都为开发智能合约带来了极大的方便。

### 三、Oraclize
下面是通过`Oraclize`获取`random.org`上随机数的最简单的智能合约：

调用了 [https://www.random.org/integers/?num=4&min=1&max=13&col=4&base=10&format=plain&rnd=new](https://www.random.org/integers/?num=4&min=1&max=13&col=4&base=10&format=plain&rnd=new)

```
pragma solidity ^0.4.11;
import "github.com/oraclize/ethereum-api/oraclizeAPI.sol";

contract RandomNumber is usingOraclize {
    
    string public randomNumber;

    function __callback(bytes32 myid, string result) public {
        require(msg.sender == oraclize_cbAddress());
        randomNumber = result;
    }
    
    function update() public payable {
        string memory url = "https://www.random.org/integers/?num=4&min=1&max=13&col=4&base=10&format=plain&rnd=new";
        oraclize_query("URL", url);
    }
}
```

合约部署后，执行`update`函数即可。

凡事可以通过链接在互联网上访问的信息，都可以通过上述方式调用到智能合约中。有些返回文本，有些返回`JSON`格式或`XML`格式。

如果是`JSON`格式的话，可以通过类似如下方式调用：

```
//ETH/USD价格：
json(https://api.gdax.com/products/ETH-USD/ticker).price

//BTC/USD价格
json(https://www.therocktrading.com/api/ticker/BTCEUR).result.0.last
```

如果是`XML`格式的话，可以通过以下方式调用：

```
//油价：
xml(https://www.fueleconomy.gov/ws/rest/fuelprices).fuelPrices.diesel
```

如果需要对调用添加参数，可以使用如下方式：

```
oraclize_query("URL", "json(https://shapeshift.io/sendamount).success.deposit",
  '{"pair":"eth_btc","amount":"1","withdrawal":"1AAcCo21EUc1jbocjssSQDzLna9Vem2UN5"}')
```
