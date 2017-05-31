# 让你的node、express服务器支持https

## http与https


HTTP: 超文本传输协议 (HTTP-Hypertext transfer protocol) 是一种详细规定了浏览器和万维网服务器之间互相通信的规则，通过因特网传送万维网文档的数据传送协议。

HTTPS:（全称：Hypertext Transfer Protocol over Secure Socket Layer），是以安全为目标的HTTP通道，简单讲是HTTP的安全版。即HTTP下加入SSL层，HTTPS的安全基础是SSL，因此加密的详细内容就需要SSL。 它是一个URI scheme（抽象标识符体系），句法类同http:体系。用于安全的HTTP数据传输。https:URL表明它使用了HTTP，但HTTPS存在不同于HTTP的默认端口及一个加密/身份验证层（在HTTP与TCP之间）。这个系统的最初研发由网景公司进行，提供了身份验证与加密通讯方法，现在它被广泛用于万维网上安全敏感的通讯，例如交易支付方面。
*
HTTPS和HTTP的区别

https协议需要到ca申请证书，一般免费证书很少，需要交费。
http是超文本传输协议，信息是明文传输，https 则是具有安全性的ssl加密传输协议。
http和https使用的是完全不同的连接方式，用的端口也不一样，前者是80，后者是443。
http的连接很简单，是无状态的；HTTPS协议是由SSL+HTTP协议构建的可进行加密传输、身份认证的网络协议，比http协议安全。

# 申请证书

**上面说到https需要得到ca申请证书，当然我们可以使用自签证，例如openSLL,当然也有免费发放证书的，这里我们向[sslforfree](https://www.sslforfree.com)申请免费证书**

**申请的流程这里稍微要注意一点的是域名验证当中的证书路径，具体流程参考[使用教学](http://www.chinaz.com/web/2016/0216/504896.shtml)**

+ 1.在主页直接输入域名
![avatar](https://raw.githubusercontent.com/hustchenshu/static_source/master/blog/images//cer_1.png)

+ 2。域名验证，这里我们选择Manual Verification
![avatar](https://raw.githubusercontent.com/hustchenshu/static_source/master/blog/images//cer-2.png)

+ 3.下载下图中的两个文件，将这两个文件放置到服务器的静态路径的跟目录，
![avatar](https://raw.githubusercontent.com/hustchenshu/static_source/master/blog/images//cer-3.png)
例如：我的服务器系统结构如下，而且在index文件当中定义了静态路径为public/pages，因此我们需要在这里进行操作，即在pages文件夹下面新建文件.well-known，然后在.well-known文件夹下面新建文件夹acme-challenge，最后将之前下载的两个验证文件放入到acme-challenge文件夹中；
![avatar](https://raw.githubusercontent.com/hustchenshu/static_source/master/blog/images//cer-4.png)
+ 4.当验证通过，打包下载给出的3个文件，如下：
![avatar](https://raw.githubusercontent.com/hustchenshu/static_source/master/blog/images//cer-5.png)
+ 5.将下载下来的3个文件上传的服务器，这里我放置在服务器根目录新建的key文件夹

# 修改启动文件

## 之前的启动文件是基于express编写的，大致如下：
```
const express = require('express');
const fs = require('fs');
const path = require('path');


const app = express();

app.use(express.static('public'));

app.use(express.static(path.resolve(__dirname, 'public/pages')));

const router_home = require('./routers/index.js');
app.use('/', router_home);

app.get('*', function(req, res){
    res.sendFile(path.resolve(__dirname, 'public/pages/404.html'), {
        title: 'No Found'
    })
});
app.listen(80);
```

## 添加https模块支持https
```
const express = require('express');
const fs = require('fs');
const path = require('path');

const http = require('http');
const https = require('https');

const app = express();

app.use(express.static('public'));

app.use(express.static(path.resolve(__dirname, 'public/pages')));

const router_home = require('./routers/index.js');
app.use('/', router_home);

app.get('*', function(req, res){
    res.sendFile(path.resolve(__dirname, 'public/pages/404.html'), {
        title: 'No Found'
    })
});
// app.listen(80);

var options = {
  ca: fs.readFileSync('key/ca_bundle.crt'),
  key: fs.readFileSync('key/private.key'),
  cert: fs.readFileSync('key/certificate.crt')
};


var httpServer = http.createServer(app);
var httpsServer = https.createServer(options, app);

httpServer.listen(80, function() {
    console.log('HTTP Server is running on: http://localhost:%s', 80);
});
httpsServer.listen(443, function() {
    console.log('HTTPS Server is running on: https://localhost:%s', 443);
});

```

# 总结
最后打开网页查看，完美支持

+ http访问

![avatar](https://raw.githubusercontent.com/hustchenshu/static_source/master/blog/images//cer-6.png)

+ https访问

![avatar](https://raw.githubusercontent.com/hustchenshu/static_source/master/blog/images//cer-7.png)





