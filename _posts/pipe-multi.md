---
title: readstream pipe到多个writestream 
---
Node的pipe很好用，比如启动个http的proxy是个很简单的事
```javascript

let httpServer = http.createServer(async (req, res) => {

    // 如果代理服务器需要认证
    // 'Proxy-Authentication': 'Base ' + new Buffer('user:password').toString('base64') // 替换为代理服务器用户名和密码
    let headers = req.headers || {};


    let proxyOpt = {
        host: options.proxy.host, // 这里是代理服务器
        port: options.proxy.port || 80, // 这里是代理服务器端口
        path: 'http://' + options.proxy.host + req.url,
        method: req.method,
        headers: headers
    };


    let proxy = http.request(proxyOpt, function(pres) {
        res.writeHeader(pres.statusCode, pres.headers);
        pres.pipe(res);
    })
    req.pipe(proxy);


});

httpServer.listen(options.port || 7000);
```

但是pipe只能消费一次，如果需求是把代理的返回内容都记录下来，但是你不能处理相同的流两次,就有点头疼了，当然可以不用pres.pipe(res)，改用下面的方式也能解决问题，但是不够漂亮 
```javascript
let result = [];
pres.on('data', (chunk) => {
    result.push(chunk);
})
pre.on('end', () => {
    console.log(result.join(''));
})
```

### 其实node本身提供了个PassThrough，它是一个简单的实现的一个Transform ###
```javascript
const evtPass = require('stream').PassThrough;
.....
let pass = new evtPass;
pass.pipe(fs.createWriteStream('./pass.txt'));

let proxy = http.request(proxyOpt, function(pres) {
    res.writeHeader(pres.statusCode, pres.headers);
    pres.pipe(pass);
    pres.pipe(res);
})
```
漂亮多了不是
