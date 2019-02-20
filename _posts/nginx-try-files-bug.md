---
title: nginx try_files的一个小坑 
---

nginx做node的反向代理，打算实现如果node挂掉了，nginx继续提供静态服务，原以为proxy_intercept_errors打开，设下error_page就好，配置如下

```nginx
    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_intercept_errors on;
        proxy_redirect off;
        error_page 500 502  @handle_error;
    }

    location  @handle_error {
        root html;
        try_files $uri $uri/ /index.html;
    }
```

  try_files $uri $uri/ /index.html @404 = 404; 是希望如果访问/abc.txt 如果html下有就返回 否则返回默认的index.html。 <br/> 

如果path有指定内容都很正常，当path 为/ 时，问题出现了，nginx报502，打开error_log，发现有两条记录:  

### http://127.0.0.1:3000/ 和 http://127.0.0.1:3000/index.html  <br/>  

  开始以为是try_files使用不当（try_files最后一个参数会做个内部转向），将配置改成：<br/>

### try_files $uri $uri/ /index.html @404 = 404;  <br/>

  依然502，尝试google也没找到解答，而且配置改成  try_files $uri $uri/ /blahblah.html @404 = 404; 依然是有个 http://127.0.0.1:3000/index.html
的错误 导致502.  <br/>

  最后灵光一现 会不会是$uri等于/ try_files时 $uri/ ==> / ==> /index.html(这个是nginx的默认文件), 修改下去掉 $uri/ 果然好了  <br/>



```nginx
    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_intercept_errors on;
        proxy_redirect off;
        error_page 500 502  @handle_error;
    }

    location  @handle_error {
        root html;
        try_files $uri /index.html /404.html = 404;
    }
```
