## 从零开始开发一个完整的以太坊猜数字游戏并在IPFS上部署

#### 编译 By 古千峰 Jacky@BTCMedia、IPFSForce

##### 本教程将完成一个以太坊游戏，该游戏的规则如下：从1-10个数字中任选一个数，100个用户下注后开奖，猜对的人，平分奖金。见下图：

![](http://images.laidingyi.com/18-8-10/76021812.jpg)

### 前言
#### 通过本教程，将学会：

* 如何从零开始创建一个智能合约

* 如何在以太坊`Ropsten`测试网络上部署一个智能合约

* 如何使用`Remix`调试工具部署并调试`Solidity`智能合约

* 如何创建一个前端Dapp应用，并用`webpack`打包

* 如何将Dapp与已部署好的智能合约连接

* 如何在IPFS网络上部署Dapp应用

该Dapp完成后，在浏览器中运行，并结合`Metamask`钱包使用。

#### 学习本教程，需要掌握以下技术和工具：

* 区块链：将在以太坊测试网络`Ropsten`上运行

* 存储：将在分布式存储`IPFS`网络上永远保存该应用

* 前端：使用`React`开发，`webpack`打包

* 智能合约语言：使用`Solidity 0.4.11`

* 智能合约调试与部署：使用在线`Remix`

* 前端智能合约连接：使用`web3.js`

* 智能合约开发框架：使用`Truffle`编译、测试并部署智能合约（辅助）

* 开发环境：`Nodejs`最新版

* 钱包：`Metamask`浏览器钱包

---
### 目录
一、配置项目

二、智能合约编程

三、创建前端应用

四、使用IPFS部署应用

---

### 一、启动项目开发环境
#### 1- 安装开发环境和需要的模块

确保系统内已经安装了`Nodejs`

创建一个新的目录：`casino-ethereum`，然后执行：

```
//安装truffle，并加入到devDependencies (-D) and globally (-g)
$ npm install -D -g truffle 

//启动truffle，下载truffle框架
$ truffle init

//生成package.json包
$ npm init -y 

//安装需要的模块，包括：webpack，react，babel，css以、json以及以太坊前端工具web3，这些也是开发以太坊前端应用的基本工具。注意：不要安装1.0.0的beta版
$ npm install -D webpack react react-dom babel-core babel-loader babel-preset-react babel-preset-env css-loader style-loader json-loader web3@0.20.0 

//安装http-server服务，以便通过localhost:3030端口访问，需要全局安装
npm i -g http-server

//安装babel-loader的预处理模块
npm i -D babel-preset-stage-2
npm i -D babel-preset-es2015
```
#### 2- 配置webpack

在目录下，新建文件`webpack.config.js`，这是打包配置文件，输入以下代码：

```
const path = require('path')

module.exports = {
   mode: 'development', //3.0版本后必须增加，生产环境是换成 production；之前webpack版本，不要使用mode
   entry: path.join(__dirname, 'src/js', 'index.js'), // 所有前端代码在 src/js/index.js 中
   output: {
      path: path.join(__dirname, 'dist'),
      filename: 'build.js'                            // 最终程序文件在 dist/build.js
   },
   module: {
      rules: [{                                       // webpack 3.0版本后，使用rules；之前版本用 loaders
         test: /\.css$/,                              // 在 react 中加载 css
         use: ['style-loader', 'css-loader'],         // 加载 css 时使用的工具
         include: /src/                               
      }, {
         test: /\.jsx?$/,                             // 加载 jsx 文件，jsx 是javascript的一种糖果文件
         loader: 'babel-loader',                      // 加载器
         exclude: /node_modules/,                     
         query: {
            presets: ['es2015', 'react', 'stage-2']
         }
      }, {
         test: /\.json$/,                             // json 文件加载
         exclude: /node_modules/,
         loader: 'json-loader'
      }]
   }
}
```

#### 3- 创建目录

然后，创建如下目录：

* 目录：`src/js`，并在该目录下创建`index.js`文件

* 目录：`src/css`，并在该目录下创建`index.css`文件

* 目录：`build`，并在该目录下创建`index.html`文件

最终的目录结构为：

```
contracts/
-- Migrations.sol
migrations/
node_modules/
test/
src/
-- css/index.css
-- js/index.js
build/
-- index.html
package.json 
truffle-config.js
truffle.js
webpack.config.js
```

#### 4- 初始化代码
将以下代码粘贴至：`build/index.html`文件中：

```
<!DOCTYPE html>
<html lang="en">
<head>
   <meta charset="UTF-8">
   <meta name="viewport" content="width=device-width, initial-scale=1.0">
   <title>以太坊开发 案例二：猜数游戏</title>
</head>
<body>
   <div id="root"></div>
   <script src="build.js"></script>
</body>
</html>
```

以上代码中，设定了id为`root`的`<div>`，用于插入React生成的代码。

另外，`webpack`生成的编译文件，放在`build.js`中。

### 二、智能合约编程
在本章中，将从零开始讲述智能合约编写，并在正式部署前调试成功。

首先，在`contracts`目录下添加文件：`Casino.sol`。

在文件头添加Solidity版本声明和程序框架：

```
pragma solidity ^0.4.20;

contract Casino {             //只有两种类型：contract和library。类名必须要与文件名完全一致
   address public owner;      //定义owner变量

   function Casino() public {
      owner = msg.sender;     //构造函数只在智能合约部署时一次性运行，将运行该合约的账户设为合约所有人owner
   }

   function kill() public {
      if(msg.sender == owner) selfdestruct(owner); //如果是合约所有人发送的kill命令，则将此合约销毁。合约销毁后，该合约中的所有代币将自动转入Owner账户。此操作只在被黑客攻击导致无法挽回时使用，但是建议每个合约都需要部署该方法。
   }
}
```

结合该项目的目的，我们需要考虑以下事情：

* 记录有多少用户已经下注，以及每个用户下注的数字

* 每注的最小下注金额（ETH）

* 总的下注金额（ETH）

* 一个记录总下注数的变量

* 合适结束下注，并开奖

* 将奖金扣除费用后，自动发放给每位中奖者

* 如果没有中奖者，则将奖金平均后返还每位参与者

下面我们将逐一讲述：

首先，创建一个玩家的`struct`数据类型，然后通过`mapping`类型定义玩家数组：

```
   struct Player {
      uint256 amountBet;        //下注金额
      uint256 numberSelected;   //选择的数字
   }

   mapping(address => Player) public playerInfo; //实例化playerInfo
```

以后，可以使用`playerInfo[用户的以太坊地址].amountBet`来获取或设置该用户的下注金额。

然后，定义一些公共变量：

```
   uint256 public minimumBet = 100 finney;//最小下注金额，0.1ETH
   uint256 public totalBet;               //总下注金额
   uint256 public numberOfBets;           //已下注人数
   uint256 public maxAmountOfBets = 100;  //最多下注人数
   address[] public players;              //玩家数组
```
这里再加两种常用的数据类型：`byte32`和`string`，这两种都是字符串，需要加双引号。

我们修改一下之前的构造函数，让初始化合约时，传入一个最小下注金额的常量：

```
function Casino(uint256 _minimumBet){
   owner = msg.sender;
   if(_minimumBet > 0 ) minimumBet = _minimumBet;
}
```
注意：在`Solidity`中，代币的单位统一为`wei`，不存在小数。一个ETH=1000...0 wei（一共有18个0），建议使用[换算计算器](https://etherconverter.online/)

接下去，添加以下代码：

```
   function bet(uint256 numberSelected) public payable {
      require(!checkPlayerExists(msg.sender));                //判断发送指令的人是否已在玩家列表中登记
      require(numberSelected >= 1 && numberSelected <= 10);   //判断选择的数字范围
      require(msg.value >= minimumBet);                       //判断玩家下注的金额是否满足最低下注额
      
      playerInfo[msg.sender].amountBet = msg.value;           //记录下注金额到数组
      playerInfo[msg.sender].numberSelected = numberSelected; //记录下注的数字到数组
      numberOfBets++;                                         //总下注次数+1
      players.push(msg.sender);                               //把用户帐号压入player数组
      totalBet += msg.value;                                  //总下注金额增加
      
      if(numberOfBets >= maxAmountOfBets) generateNumberWinner(); //判断下注的人次是否超过了100个
   }
```

该代码是用户下注时执行，其中：

* `msg.sender` 是发送者帐号，`msg.value` 是发送者金额。

* `payable` 是一个`modifier`，函数修改器，用在智能合约中进行一些函数功能行为的修改，例如对函数执行前置条件的自动检查，有点像条件函数，或者事件触发器。

在这里，所有涉及支付的函数，都需要设定为`payable`，如果没有该`modifier`函数，打到合约的代币会被自动退回。

常见的`modifier`函数还有`onlyOwner`，表示只有owner才能执行该函数，代码如下：

```
    modifier onlyOwner {
        require(msg.sender == owner) _;
    }
```

* `require()`函数用来判断括号中的条件，如果是`True`，则继续，否则，将用户的代币返回给用户账户，函数结束。与0.4.10版本前`Solidity`使用`if...throw`一样，但代码更整洁。类似的还有`assert()`，只是`assert()`对于错误的操作会扣除Gas，`require()`不会。

* `checkPlayerExists`函数用来判断用户是否存在，代码如下：

```
   function checkPlayerExists(address player) public constant returns(bool){
      for(uint256 i = 0; i < players.length; i++){
         if(players[i] == player) return true;
      }
      return false;
   }
```

以上函数中的返回值类型为`constant`，是指直接返回某个值，而无需改变账户状态，这个操作无需消耗Gas。

一般`constant`会和`returns()`返回值定义，一起使用。

* 上面代码的最后一句，判断下注人数如果超过100，则执行开奖函数`generateNumberWinner`：

```
    function generateNumberWinner() public {
        uint256 numberGenerated = block.number % 10 + 1;
        distributePrizes(numberGenerated);       //执行奖金分发函数
    }

```

这里使用了不安全的随机数获取方法，即当前块的高度`block.number`，并取其个位数加1，该方法只能用于教学，不能用于实际使用。因为很容易被猜到。

* 随机数产生是区块链智能合约编程中的一个重点和难点，建议可以使用第三方`Oracle`随机数种子，如：[Oraclize](http://www.oraclize.it/)，[ChainLink](https://www.smartcontract.com/link), [ZAP](https://www.zap.org/) 

#### 本教程原作者已经在完整版中，将Oraclize工具生成真实的随机数，[点击这里](https://github.com/merlox/casino-ethereum/blob/master/contracts/Casino.sol)获取完整代码。

* 实现奖金分发函数

```
    function distributePrizes(uint256 numberWinner) public {
        address[100] memory winners;               //保存获胜者的临时数组，memory类型的数据在函数执行完毕后释放，这类临时变量必须是固定长度，本例中定义长度为100，即最多全部猜中情况下，获胜者不会超过100名
        uint256 hasWiner = 0;                      //用来判断是否有赢家
        uint256 count = 0;                         // 因为winners数组指定长度，所以无法通过length获取获奖人数，只能设这个变量

        for (uint256 i = 0; i < players.length; i++) {
            address playerAddress = players[i];
            if (playerInfo[playerAddress].numberSelected == numberWinner) {
                //如果玩家选的数字等于随机生成的获奖数字，则将获奖者压入到数组
                winners[count] = playerAddress;
                count++;
            }
            delete playerInfo[playerAddress];      // 无论是否获奖，都将释放玩家数据，不做保存
        }

        if (count == 0) {
            //如果没有人猜中
            count = maxAmountOfBets;
            hasWiner = 0;
        } else {
            hasWiner = 1;
        }

        uint256 winnerEtherAmount = totalBet * 95 / 100 / count; // 系统抽取5%

        for (uint256 j = 0; j < count; j++) {
            if (hasWiner == 1) {
                //如果有人猜中，则分给猜中的人
                if (winners[j] != address(0)) winners[j].transfer(winnerEtherAmount);
            } else {
                //如果没有人猜中，则分给所有竞猜的人
                if (players[j] != address(0)) players[j].transfer(winnerEtherAmount);
            }
        }

        players.length = 0; // Delete all the players array
        totalBet = 0;
        numberOfBets = 0;
    }
```

#### 到此为止，合约部分全部完成，接下去通过`Remix`IDE工具进行调试。调试前，需要在浏览器中装好`Metamask`钱包软件

##### 1- 准备测试账户
在`Metamask`钱包的`Rospten`测试网络中新建一个账户，在浏览器中打开以太坊水龙头工具[https://faucet.metamask.io/](https://faucet.metamask.io/)，点击绿色按钮，就可以免费获得测试用的以太币。

![](http://images.laidingyi.com/18-8-8/9851782.jpg)

##### 2- 配置Rimix环境
打开[Remix网站](http://remix.ethereum.org/)，将以上代码复制到`Remix`调试工具中，并刷新`Remix`网页。

点击右上角`Run`标签，注意在`Environment`一栏中，选择:`Injected Web Rospten`。然后会在`Account`一栏中看到`Metamask`钱包中的账户名，如下图：

![](http://images.laidingyi.com/18-8-8/52107652.jpg)

##### 3- 编译智能合约
点击右上角`Compile`标签，点击`Start to Comile`开始编译。如果一切正常，编译通过。

##### 4- 部署智能合约
回到`Run`标签，点击`Deploy`按钮，开始部署合约。正常情况，会跳出`Metamask`钱包，输入`Gas`费用(注意，千万不要是0)，点击`Submit`，在console窗口中会提示`transaction`的区块链链接地址，过一会，会提示部署成功的信息，然后点击`Remix`监控窗口提示的链接，点击打开浏览器，查看部署合约的交易，并获取合约地址。

注意：请务必`将合约地址复制到记事本`。

至此合约部署完毕。

在部署过程中，经常会提示错误，不要强行发送`transaction`，仔细查看代码，每个很小的bug，都可能会让程序无法正常运行，并提示报错。

每次修改bug，都需要重新部署合约，一个成熟的合约，要不厌其烦的反反复复检查代码、优化代码。

##### 5- 使用合约
将复制的智能合约地址，复制到`at Address`文本框中，点击按钮调用该合约。

使用每个`public`函数，以及所有`public`变量。

##### 其他：
*使用javascript部署合约*

在`migrations`目录下，新建一个文件`2_deploy_contracts.js`，内容如下：

```
var Casino = artifacts.require("./Casino.sol");

module.exports = function(deployer) {
  //前面两个参数分别对应合约构造函数中的两个参数，gas为部署合约时的Gas费用
  deployer.deploy(web3.toWei(0.1, 'ether'), 100, {gas: 3000000}); 
};
```

*使用truffle部署智能合约*

除了`Remix`调试工具之外，`Truffle`也有一套部署工具。运行`truffle compile`编译源码，编译成功后，在`build/contracts/`目录下，会生成`Casino.json`文件。

### 三、创建前端应用
#### 1- 创建主入口代码
打开`src/js/index.js`文件，加入以下代码：

```
import React from 'react'
import ReactDOM from 'react-dom'
import Web3 from 'web3'
import './../css/index.css'

class App extends React.Component {
   constructor(props){
      super(props)
      this.state = {
         lastWinner: 0,
         timer: 0
      }
   }

voteNumber(number){
      console.log(number)
   }

render(){
      return (
         <div className="main-container">
            <h1>Bet for your best number and win huge amounts of Ether</h1>

<div className="block">
               <h4>Timer:</h4> &nbsp;
               <span ref="timer"> {this.state.timer}</span>
            </div>

<div className="block">
               <h4>Last winner:</h4> &nbsp;
               <span ref="last-winner">{this.state.lastWinner}</span>
            </div>

<hr/>

<h2>Vote for the next number</h2>
            <ul>
               <li onClick={() => {this.voteNumber(1)}}>1</li>
               <li onClick={() => {this.voteNumber(2)}}>2</li>
               <li onClick={() => {this.voteNumber(3)}}>3</li>
               <li onClick={() => {this.voteNumber(4)}}>4</li>
               <li onClick={() => {this.voteNumber(5)}}>5</li>
               <li onClick={() => {this.voteNumber(6)}}>6</li>
               <li onClick={() => {this.voteNumber(7)}}>7</li>
               <li onClick={() => {this.voteNumber(8)}}>8</li>
               <li onClick={() => {this.voteNumber(9)}}>9</li>
               <li onClick={() => {this.voteNumber(10)}}>10</li>
            </ul>
         </div>
      )
   }
}

ReactDOM.render(
   <App />,
   document.querySelector('#root')
)
```

#### 2- 创建CSS文件
打开`src/css/index.css`文件，加入以下代码：

```
body {
    font-family: 'open sans';
    margin: 0;
}

ul {
    list-style-type: none;
    padding-left: 0;
    display: flex;
}

li {
    padding: 40px;
    border: 2px solid rgb(30, 134, 255);
    margin-right: 5px;
    border-radius: 10px;
    cursor: pointer;
}

li:hover {
    background-color: rgb(30, 134, 255);
    color: white;
}

li:active {
    opacity: 0.7;
}

* {
    color: #444444;
}

.main-container {
    padding: 20px;
}

.block {
    display: flex;
    align-items: center;
}

.number-selected {
    background-color: rgb(30, 134, 255);
    color: white;
}

.bet-input {
    padding: 15px;
    border-radius: 10px;
    border: 1px solid lightgrey;
    font-size: 15pt;
    margin: 0 10px;
}
```

#### 3- 用webpack打包应用并运行

在命令行中，使用命令：`webpack`或者`npm run build`编译打包。

以上命令在`package.json`文件中配置，查看`scripts`，如下：

```
  "scripts": {
    "build": "webpack --log-level=debug",                  //npm run build等同于webpack命令
    "start": "webpack-dev-server  --port 3030  --inline --content-base ./build"
  },
```
然后运行：`npm start`启动web服务，默认情况下`npm init`时会生成`8080`端口的web服务，如果冲突，可以改为其他端口。如本例改为了`3030`本地端口。

接着在浏览器中打开: `http://127.0.0.1:3030`，可以看到如下网页：

![](http://images.laidingyi.com/18-8-9/28237485.jpg)

#### 4- 连接智能合约与javascript前端
在`Remix`中部署合约，找到ABI文件，并复制。

在`index.js`中添加如下代码：

```
        if (typeof web3 != "undefined") {
            //启动Metamask
            console.log("Using web3 detected from external source like Metamask")
            this.web3 = new Web3(web3.currentProvider)
        } else {
            //启用本地以太坊网络或者Truffle的Ganache
            this.web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8545"))
        }

        const contractAddress = "0xB2bE09289F9f7103964f57aAF04119fcB79d2149" //本案例部署的合约帐号，可换
		const abi = 合约的ABI数组，一个非常长的数组
        const MyContract = web3.eth.contract(abi)
        this.state.ContractInstance = MyContract.at(contractAddress)
```

执行合约中函数的基本方法，以合约中的`bet`函数为例：

```
yourContractInstance.bet(7, {                             // 7为函数参数，即下注的数字
   gas: 300000,                                           // Gas
   from: web3.eth.accounts[0],                            // 用户帐号，accounts是数组，取第一个元素
   value: web3.toWei(0.1, 'ether')                        // 发送金额，单位wei
}, (err, result) => {...})
```

如果调用不需要Gas的方法或者变量，使用如下代码：

```
yourContractInstance.maxAmountOfBets((err, result) => {
   if(result != null) {...}
})
```

在`index.js`继续添加以下代码：

```
    componentDidMount() {
        this.updateState()
        this.setupListeners()

        setInterval(this.updateState.bind(this), 10e3)
    }

    updateState() {
        this.state.ContractInstance.minimumBet((err, result) => {
            if (result != null) {
                this.setState({
                    minimumBet: parseFloat(web3.fromWei(result, 'ether'))
                })
            }
        })
        this.state.ContractInstance.totalBet((err, result) => {
            if (result != null) {
                this.setState({
                    totalBet: parseFloat(web3.fromWei(result, 'ether'))
                })
            }
        })
        this.state.ContractInstance.numberOfBets((err, result) => {
            if (result != null) {
                this.setState({
                    numberOfBets: parseInt(result)
                })
            }
        })
        this.state.ContractInstance.maxAmountOfBets((err, result) => {
            if (result != null) {
                this.setState({
                    maxAmountOfBets: parseInt(result)
                })
            }
        })
    }

    // 设置监听器
    setupListeners() {
        let liNodes = this.refs.numbers.querySelectorAll('li')
        liNodes.forEach(number => {
            number.addEventListener('click', event => {
                event.target.className = 'number-selected'
                this.voteNumber(parseInt(event.target.innerHTML), done => {

                    // Remove the other number selected
                    for (let i = 0; i < liNodes.length; i++) {
                        liNodes[i].className = ''
                    }
                })
            })
        })
    }

	// 下注
    voteNumber(number, cb) {
        let bet = this.refs['ether-bet'].value

        if (!bet) bet = 0.1 //默认下注为0.1ETH

        if (parseFloat(bet) < this.state.minimumBet) {
            alert('You must bet more than the minimum')
            cb()
        } else {
            this.state.ContractInstance.bet(number, {
                gas: 300000,
                from: web3.eth.accounts[0],
                value: web3.toWei(bet, 'ether')
            }, (err, result) => {
                cb()
            })
        }
    }
    
```

至此，主要的程序已经完成，`webpack`打包编译，运行`npm start`，然后在浏览器中打开：`http://127.0.0.1:3030`可以试玩。

### 四、使用IPFS部署应用
在本章，我们将看到IPFS的强大，她可以方便的部署一个去中心化的应用。

启动IPFS网络后，运行以下命令：

```
ipfs add -r dist/
ipfs name publish 上面生成的Hash值
```
之后就可以直接用：`http://网关/ipfs/网站Hash` 进行访问。

比如本教程示例在：[http://eternum.io/ipfs/QmfZrt27ohzMZfd6jjXxhvze6ZaLo9e8keRsoTBsdEjfcY](http://eternum.io/ipfs/QmfZrt27ohzMZfd6jjXxhvze6ZaLo9e8keRsoTBsdEjfcY)

即可在IPFS网络上访问并运行该分布式应用。

好了，本教程到此结束，感谢该教程的原作者[Merunas](https://medium.com/@merunasgrincalaitis?source=post_header_lockup)，感谢Satoshi、感谢Vitalic、感谢Daniel，让我们能够在一个完全去中心化的环境中如此方便的开发竞猜类应用:)

----
#### 参考文档和常用工具：
1- [教程原文](https://medium.com/@merunasgrincalaitis/the-ultimate-end-to-end-tutorial-to-create-and-deploy-a-fully-descentralized-dapp-in-ethereum-18f0cf6d7e0e)

2- [完整的智能合约源码](https://github.com/merlox/casino-ethereum/blob/master/contracts/Casino.sol)

3- [以太坊web3中文文档](http://web3.tryblockchain.org/)

4- [Solidity在线调试工具](http://remix.ethereum.org/)

5- [以太坊单位换算器](https://etherconverter.online/)

6- [Ropsten测试网络免费获取以太币]()

7- [Solidity文档](http://solidity.readthedocs.io/en/develop/)

8- [Web3.js文档](https://github.com/ethereum/wiki/wiki/JavaScript-API)

