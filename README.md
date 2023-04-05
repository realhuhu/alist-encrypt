# Alist-encrypt

这个项目主要是对 alist 的服务进行代理，提供 webdav 的加解密功能。支持 alist 网页在线播放加密的视频，查看加密的图片等功能，同时在 webdav 下的操作透明，自动实现文件资源的加解密。

到目前为止，此项目的算法已经基本确定下来了，用 RC4 更安全，之前的 mix 混淆明文的方案其实也是可行的，但是作为加密的话，安全强度还不够，对付网盘当然是足够了。而 RC4 更加安全，由于使用 nodejs 进行实现，但是性能会稍微差一些（但是依然非常的高效，电视盒子的性能可以跑满 100M 带宽没问题，主要吃单核性能），mix 混淆的方案是基本没有性能的开销。

现阶段还没有完全可用于生产，切记不要直接大量上传加密文件，因为算法还没最终确定下来。一定不要大量上传加密文件，否则到时候无法使用生产版本就那些解密。

算法确定下来后，接下来就可以正常实现业务了，release 版本 应该很快就可以和大家见面了。体验版可以用 docker 进行体验。

因为项目前期会有很多 bug，issue 处理的讨论不太方便，讨论反馈群：422035582 ，里面也会有操作说明。

## 一、需求背景

AList 是一个支持多种存储、云网盘，支持网页浏览和提供 WebDAV 服务的应用程序。最近的阿里云盘很火，因为不限速，所以不少人使用阿里云盘配合 alist 当做个人的影院，随时在线观看视频。

国内的云盘有很多，除了阿里云盘还有天翼云盘也是不限速的，但是几乎都存在一个问题，敏感资源会被删除，相信很多人经历文件被删除掉的噩梦。那么有没有什么办法可以避免这样的问题呢，最简单的方案就是加密后上传。那么就有大局限性，不能实时在线播放视频，当然也有一些方案可以做到。加密后的文件分享也存在一定的不方便（密码不方便对外提供）。

Alist-encrypt 就是为了解决这个问题，它可以结合 webdav 服务器进行使用，在文件上传的时候进行加密，文件下载的时候进行解密，由于使用的是流加密方案，所以可以很轻松实现在线播放视频，浏览图片、文件等。目前主流的方案都是使用 alist 来实现网盘 webdav 的服务，所以 Alist-encrypt 支持 alist 服务，并且优先支持它的适配，支持网页版在线播放视频等功能。

关于这个项目的使用场景，对文件安全隐私有一定的需求，防止云盘扫描删除，有实时播放视频和下载的需求。

## 三、实现原理

项目的实现比较简单，原始孵化的项目地址在这里：[tlf-encryption](https://github.com/traceless/tlf-encryption) 它有描述 mix 混淆算法的实现，代理服务的实现思路，也有基础版本的代码实现，可以学习参考。mix 的方式作为混淆文件没有问题，如果当做是加密的方案，那么强度可能不够，通过文件的特征，容易暴力破解。

当前项目算法已经换成成熟的方案-RC4 算法，安全性大大的提高了，理论上安全强度还是有点瑕疵，但已经是可以忽略不计了。据研究 10 亿次的加密才会有几率出现弱密匙，而且即便出现弱密匙，也只能看到几十个字节，所以安全性完全不是问题。

## 二、安装使用

### 下载运行

需要先安装 nodejs 的环境，具体安装方法，请参考网上的教程

1、下载此项目，进入 node-proxy 目录执行

> npm i

2、修改 conf/config.js 配置文件，添加 alist 服务地址端口，添加 alist 的网盘中需要进行加密的文件夹路径。

3、然后执行启动命令

> node app.js

最后就打开代理服务器地址 http://127.0.0.1:5344/public/index.html 即可进入配置页面，账号 admin，密码默认 123456。配置后之后，打开http://127.0.0.1:5344 即可访问到 alist 的服务了

### docker 安装

运行拉取镜像命令

> docker pull prophet310/alist-encrypt:beta

执行启动容器即可

> docker run -d -p 5344:5344 -v /etc/conf:/node-proxy/conf --name=alist-encrypt prophet310/alist-encrypt:beta

arm 版本目前单独打包 beta-arm，后续再放一起

> docker run -d -p 5344:5344 -v /etc/conf:/node-proxy/conf --name=alist-encrypt prophet310/alist-encrypt:beta-arm

启动后就打开代理服务器地址 http://127.0.0.1:5344/public/index.html 即可进入配置页面，账号 admin，密码默认 123456。配置后之后，打开http://127.0.0.1:5344 即可访问到 alist 的服务了。

对于路径的设置，目前是支持正则表达式的，推荐表达式例如: movie_encrypt/\* ，这样的话所有的movie_encrypt目录的文件都会被加密传输。

### 操作使用

1、alist 原本网页上的所有的操作都可以正常使用，因为 Alist-encrypt 它是透明代理，所以你所有的操作请求都是透传到 alist 上的，除了某些需要加密上传的操作和在线解密播放的操作。

2、你可以在 webdav 客户端上进行文件上传，如果设置了加密的文件夹目录，那么上传的文件就会被加密，在云盘上下载后会无法打开。但是你使用 Alist-encrypt 代理的 alist 服务还是一样可以正常下载查看，在线播放视频，查看图片等，不管是在 webdav 还是网页上都是正常使用。

3、界面上对于路径的设置，目前是支持正则表达式的，推荐表达式例如: movie_encrypt/\* ，这样的话所有的movie_encrypt目录的文件都会被加密传输。

## 四、已支持&待完善

### 已支持的功能

1. 支持 alist 网页在线播放加密的视频，查看图片，在线下载等。
2. 支持 alist 网页跳转到 IINA，VLC，Infuse 等播放器上进行播放。
3. 在 webdav 客户端上的所有操作不会受到影响，自动加解密，可播放视频、查看图片。
4. 据文件夹名字自动解密别人分享的内容。
5. 设置不同目录不同密码。
6. 提供 cli 程序进行文件解密\加密，用于分享对方在下载后解密。

### 待实现功能

1. 可以把未加密(或已加密)的文件夹 A（或文件） -> 转存到加密文件夹 B 中，用于转存别人分享的文件。
2. 可选择加密文件名。
3. 后续还会移植到 Autojs 上运行，

### 已知问题

- ~~加密文本还不能在线看，当前建议直接下载看~~ 已经支持。
- 阿里云盘无法使用 Aliyun Video Previewer 进行播放，请选择 Video 方式播放、

### 局限性

- ~~目前的实现还很基础，RC4 算法还有一点点小缺点，对于大文件的播放时，如果点击播放尾部片段就会卡几秒，这个后面会优化下~~ 已经优化
- chacha20 算法已经添加，对于非视频的资源性能更好，安全性也 ok。如果是要存储视频的，那么只建议存储 2G 以下的，否则跳转播放会有点慢，目前技术上没法解决，如果是其他的语言也许还能解决，nodejs 搞不了，重写后的算法极其慢。

## 五、FAQ

### 1、为何不使用 ASE 对称加密算法

- 因为对称加密是块加密，理论上对齐数据块是能实现的流加密的，但是需要知道文件的长度。在线播放的时候，跳转播放位置时，也需要对齐字节才能正常播放，加密和解密的业务实现比较复杂一些，实现起来头大，还有类似的大文件分片下载等处理。如果你对文件安全性要求很高，那么并不推荐这项目来加密你的数据。RC4 是比较合适一点，目前已经支持。chacha20 也支持了，但是对于视频类的文件，任然不推荐使用。AES 也有流加密算法模式，但是跟 chacha20 一样，痛点依旧存在，对视频播放的需求并不合适。有 chacha20 就不会再选择 AES 了。

设计这个项目的初衷本身就是为了躲避云盘的扫描的，文件是可以分享给对方查阅的。AES 加密或非对称加密更适合私人资料使用。对文件的安全要求不高的情况下，推荐使用这项目，它可以让你轻松躲过网盘的文件审核，也可以让你轻松找回密码（有一个原文件即可）。

**另外一种加密技巧**

- 基于这个算法的调整钥匙生成方式，可以根据文件长度 实现不同文件加密的钥匙不一样，这个后续再考虑是否有实现的必要。因为会导致无法找回文件夹密码了，同样也不方便分享整个文件夹了，得不偿失，还要考虑是否有其他使用的影响。
- 当前项目已添加 RC4 算法，安全性基本是无可挑剔了。

### 2、代理服务器性能

目前还没测试它实际的性能情况，虽然是使用 nodejs 实现的，但是它的算法确实很简单，应该还没压缩算法复杂，理论上是没多少损失的。目前使用的电视盒子 s905l3a 测试理论是可以跑满 300-500M 带宽的，证明这个代码的性能是基本满足使用的。

### 3、会考虑其他语言的实现吗？

暂时不会考虑。因为这个项目比较简单，无论是使用 go，nodejs 应该都是相当容易现实的，它的业务不会膨胀到需要用 java 语言来实现。对于性能的要求也不一定要用到 go 来支持。选 nodejs 的原因是 go 语言我并不是特别熟悉，而且 nodejs 在 web 开发效率上并不输 go 语言。后续还会移植到 Autojs 上运行，也就是安卓 app 也能直接支持，目前安卓手机可以使用 termux 进行部署。

目前 nodejs 也支持 pkg 打包，项目已经支持打包成可执行文件。

### 4、项目后续的安排

当前版本还不具备生产使用的条件，只是基本实现部分功能，还有很多网盘没测试，预计再迭代 2 个版本才能看到更完整的功能。此项目会关注 alist 那边的更新，持续适配 alist，也希望大家多多支持，提些建议。
