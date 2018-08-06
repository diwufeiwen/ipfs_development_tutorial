## 开发基于IPFS的文件分享系统

### 作者：古千峰 Jacky@BTCMedia、IPFSForce

#### 本教程目的：
通过实例分析，讲解如何基于javascript的ipfs-api组件，开发一个文件分享系统。

#### 一、环境搭建
1- 安装`ipfs-api`

```
npm install ipfs-api
```

2- 安装`nodejs`和`npm`

#### 二、下载应用源码
从IPFS网络下载：[下载地址](https://www.eternum.io/ipfs/QmdzjGqHfUa7hK4gQq4G6U4XpLZ3iBMHvBhQWaktTpBYVh)

#### 三、安装
将源码压缩包解压，进入解压后的目录`upload-file-via-browser\`中，执行：

```
npm install   //安装模块
npm start     //启动
```

#### 四、试运行
启动`ipfs`服务，执行以下命令：

```
ipfs daemon &
```
在浏览其中打开：[http://localhost:3000](http://localhost:3000)

这是，可能会发生跨域错误，分别执行以下命令：

```
ipfs config --json API.HTTPHeaders.Access-Control-Allow-Methods '["PUT", "GET", "POST", "OPTIONS"]'
ipfs config --json API.HTTPHeaders.Access-Control-Allow-Origin '["*"]'
ipfs config --json API.HTTPHeaders.Access-Control-Allow-Credentials '["true"]'
ipfs config --json API.HTTPHeaders.Access-Control-Allow-Headers '["Authorization"]'
ipfs config --json API.HTTPHeaders.Access-Control-Expose-Headers '["Location"]'
```
然后执行以下两条命令：

```
ipfs config Addresses.API /ip4/127.0.0.1/tcp/5001
ipfs daemon &
```

退出npm服务重新启动`npm start`，打开[http://localhost:3000](http://localhost:3000)，应该就可以执行了。

#### 五、代码解析
核心代码是：`\src\Api.js`，下面一条一条解读：

```
const React = require('react')       //加载React库
const ipfsAPI = require('ipfs-api')  //加载ipfs-api库

class App extends React.Component {
  constructor () {
    super()
    this.state = {
      added_file_hash: null                           //添加文件的hash值
    }
    this.ipfsApi = ipfsAPI('localhost', '5001')       //在构造函数中实例化ipfsAPI

    // bind methods
    this.captureFile = this.captureFile.bind(this)
    this.saveToIpfs = this.saveToIpfs.bind(this)
    this.handleSubmit = this.handleSubmit.bind(this)
  }

  captureFile (event) {                               //当文件组件发生变化时
    event.stopPropagation()
    event.preventDefault()
    const file = event.target.files[0]                //获取文件对象
    let reader = new window.FileReader()              //定义一个文件读取对象FileReader
    reader.onloadend = () => this.saveToIpfs(reader)  //设定reader事件，当reader读写完毕后，保存到ipfs网络
    reader.readAsArrayBuffer(file)                    //开始读取
  }

  saveToIpfs (reader) {                               //保存到ipfs网络
    let ipfsId
    const buffer = Buffer.from(reader.result)         //从reader对象中读取数据
    this.ipfsApi.add(buffer, { progress: (prog) => console.log(`received: ${prog}`) })
      .then((response) => {                           //执行添加到ipfs *****这是关键代码
        console.log(response)
        ipfsId = response[0].hash                     //hash值保存在response[0].hash中
        this.setState({added_file_hash: ipfsId})      //设置全局状态值
      }).catch((err) => {
        console.error(err)
      })
  }

  handleSubmit (event) {
    event.preventDefault()
  }

  render () {
    return (
      <div>
        <form id='captureMedia' onSubmit={this.handleSubmit}>
          <input type='file' onChange={this.captureFile} />
        </form>
        <div>
          <a target='_blank'
            href={'https://www.eternum.io/ipfs/' + this.state.added_file_hash}>
            {this.state.added_file_hash}
          </a>
        </div>
      </div>
    )
  }
}
module.exports = App
```

#### 进阶
* 添加二维码，在上传后，可以通过二维码立即分享获得文件。
* 与ETH或者EOS结合，可以做成一个收费的云文件系统。
