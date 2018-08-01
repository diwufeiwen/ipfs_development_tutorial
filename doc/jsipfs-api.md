## IPFS开发教程：js-ipfs
#### Edit by 古千峰 JackyGu@BTCMedia、IPFSForce on Aug.1,2018

### 一、安装

```
npm install ipfs --save
```
如果需要在命令行下操作，使用以下方式安装

```
npm install ipfs --global
```

npm版本 > 3.0.0

nodejs版本 > 6.0.0

如果安装时出现`EACCESS`错误，可以在`npm install`时使用`--unsafe-perm=true`或者`--allow-root`参数强制安装。

### 二、引用`js-ipfs`库

```
const IPFS = require('ipfs')
const node = new IPFS()
```

### 三、在浏览其中引用`js-ipfs`
在`html`文件中在`<script>`标签内包含以下任意一行命令即可。

```
https://unpkg.com/ipfs/dist/index.min.js

https://cdn.jsdelivr.net/npm/ipfs/dist/index.min.js

https://unpkg.com/ipfs/dist/index.js

https://cdn.jsdelivr.net/npm/ipfs/dist/index.js
```

### 四、使用命令行工具
如果安装时，使用了`--global`，则可以使用以下命令进行命令行操作。

```
jsipfs --help
```

`js-ipfs`的默认设置中做了以下设定，以防和节点运行的`go-ipfs`发生冲突。

* 默认位置: ~/.jsipfs (can be changed with env variable IPFS_PATH)
* 默认swarm端口: 4002
* 默认API端口: 5002
* 默认情况下 Bootstrap 关闭, 可以通过 `set IPFS_BOOTSTRAP=1` 开启

### 五、以进程方式运行`js-ipfs`


### 六、以模块方式运行`js-ipfs`
```
// Create the IPFS node instance
const IPFS = require('ipfs')
const node = new IPFS()

node.on('ready', () => {
  // Your node is now ready to use \o/

  // stopping a node
  node.stop(() => {
    // node is now 'offline'
  })
})
```

### 七、API
#### 1- 构造函数
一般不用

```
const IPFS = require('ipfs')
option = {
	repo: '/var/ipfs/data',         //repo数据目录，默认为~/.jsipfs
	init: true,                     //是否启动新的节点，如果已经运行过该命令，建议不要设为true
	start: true,                    //如果设为true，则自动启动节点，否则使用 node.start()手动启动节点
	pass: 'password' ,              //用于对key加密
	EXPERIMENTAL: {
		pubsub: false,              //激活  libp2p pub-sub
		sharding: false,            //是否使用分片技术，如果true，则会生成多个DAG节点
		dht: false,                 //是否使用KadDHT，即Kademlia DHT技术
	},
	config: {
		                            //用于修改IPFS节点配置，默认的文件见下面注释
	},
	libp2p: {
		                            //用于向libp2p中添加定制模块，不常用
		modules: {
			transport: [],
			peerDiscovery: []
		},
		config: {
			peerDiscovery: {
			
			}
		}
	}
}

const node = new IPFS(option)
```

注：

* [IPFS节点配置文件参考](https://github.com/ipfs/js-ipfs/tree/master/src/core/runtime/config-nodejs.js)
* [IPFS浏览器配置文件参考](https://github.com/ipfs/js-ipfs/tree/master/src/core/runtime/config-nodejs.js)

#### 2- 事件
IPFS采用事件侦听机制触发。

##### 出错时触发

```
node.on('error', error => {
  console.error(error.message)
})
```

##### 启动节点成功后触发

```
node.on('ready', () => {
	//运行命令
})
```

##### 初始化节点后触发

```
node.on('init', () => {
	//运行命令
})
```
如果在`option`中设置：`init: false`，则不会触发此事件。

##### 节点开始侦听连接时触发

```
node.on('start', () => {
	//运行命令
})
```
如果在`option`中设置：`start: false`，则不会触发此事件。

##### 停止时触发

```
node.on('stop', () => {
	//运行命令
})
```
此事件一般是执行了`node.stop()`后触发。

* 以上事件执行顺序：init -> start -> ready，可运行以下代码来查看执行顺序：

```
// Create the IPFS node instance
const IPFS = require('ipfs')
const node = new IPFS()
node.on('ready', () => {
    // Your node is now ready to use \o/
    console.log('Node is ready!')
    // stopping a node
    node.stop(() => {
        // node is now 'offline'
    })
})
node.on('error', error => console.error('Something went terribly wrong!', error))
node.on('start', () => console.log('Node started!'))
node.on('init', () => console.log('Node initialized!'))
node.on('stop', () => console.log('Node stoped!'))
```

#### 3- 方法

#### 启动节点
分别有三种模式：

第一种：直接启动

```
const node = new IPFS({ start: false })

node.start()
  .then(() => console.log('Node started!'))
  .catch(error => console.error('Node failed to start!', error))

```

第二种：使用回调函数

```
const node = new IPFS({ start: false })

node.start(error => {
  if (error) {
    console.error('Node failed to start!', error)
    return
  }
  console.log('Node started!')
})
```

第三种：使用事件

```
const node = new IPFS({ start: false })

node.on('error', error => console.error('Something went terribly wrong!', error))
node.on('start', () => console.log('Node started!'))
node.start()
```

#### 停止节点
```
const node = new IPFS()
node.on('ready', () => {
  console.log('Node is ready to use!')

  // Stop with a promise:
  node.stop()
    .then(() => console.log('Node stopped!'))
    .catch(error => console.error('Node failed to stop cleanly!', error))

  // OR use a callback:
  node.stop(error => {
    if (error) {
      console.error('Node failed to stop cleanly!', error)
      return
    }
    console.log('Node stopped!')
  })

  // OR use events:
  node.on('error', error => console.error('Something went terribly wrong!', error))
  node.stop()
})
```

### 八、添加文件到IPFS网络
使用`async`异步处理类。

相当于使用 `ipfs add` 和 `ipfs cat` 命令

```
'use strict'

const series = require('async/series')
const IPFS = require('ipfs')

const node = new IPFS()
let fileMultihash  //文件Hash值

series([
    (cb) => node.on('ready', cb),
    (cb) => node.version((err, version) => {
        if (err) { return cb(err) }
        console.log('Version:', version.version)
        cb()
    }),
    (cb) => node.files.add({
        path: 'hello.txt',                       //文件名
        content: Buffer.from('Hello World 101')  //内容
    }, (err, filesAdded) => {
        if (err) { return cb(err) }

        console.log('\nAdded file:', filesAdded[0].path, filesAdded[0].hash)
        fileMultihash = filesAdded[0].hash
        cb()
    }),
    (cb) => node.files.cat(fileMultihash, (err, data) => {
        if (err) { return cb(err) }

        console.log('\nFile content:')
        process.stdout.write(data)               //查看文件内容
    })
])
```

### 九、从IPFS网络下载文件
相当于使用 `ipfs get` 命令

```
const series = require('async/series')
const IPFS = require('ipfs')

const node = new IPFS()
const cid = 'QmccqhJg5wm5kNjAP4k4HrYxoqaXUGNuotDUqfvYBx8jrR' //获取该hash值的文件

series([
    (cb) => node.on('ready', cb),
    (cb) => node.files.get(cid, (err, data) => {
        data.forEach((file) => {
            console.log(file.path)
            //console.log(file.content.toString('utf8'))
        })
    })
])
```

### 目前只有files类比较成熟，[点击这里查看API文档](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/FILES.md)
### `js-ipfs`示例见[https://github.com/ipfs/js-ipfs/blob/master/examples](https://github.com/ipfs/js-ipfs/blob/master/examples)

