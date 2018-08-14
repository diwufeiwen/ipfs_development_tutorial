## 基于HTML5的IPFS播放器

### By 古千峰 Jacky@BTCMedia、IPFSForce

```
<video id="video" controls="" preload="none" poster="http://media.w3.org/2010/05/sintel/poster.png">
	<source id="mp4" src="https://www.eternum.io/ipfs/QmU1GSqu4w29Pt7EEM57Lhte8Lce6e7kuhRHo6rSNb2UaC" type="video/mp4">
    <p>Your user agent does not support the HTML5 Video element.</p>
</video>
```

其中 `poster` 是视频封面图片，用 `png` 格式。

`src` 是IPFS视频地址。

类 `video` 的属性和动作：
* `load()`              - 加载视频文件

* `play()`              - 播放视频文件

* `pause()`             - 暂停播放

* `currentTime+=10`     - 前进10秒，或者倒退

* `playbackRate++`      - 加倍速或者减倍速

* `playbackRate+=0.1`   - 加速或者减速

* `volume+=0.1`         - 音量加大，或者减少

* `muted=true`          - 静音，或者false，打开声音

css文件：

```
video {
	border: 1px solid black;
	padding: 0;
	margin: 0;
	width: 500px;
	height: 336px;
	background-color: black;
	margin: auto;
	display: block;
}
```
