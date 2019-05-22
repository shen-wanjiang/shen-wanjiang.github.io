---
layout:     post
title:      不同设备间文件传输利器之http-server
subtitle:   开启简单http服务器
date:       2019-05-22
author:     WJ
header-img: img/post-bg-python-unicode.jpg
catalog: true
tags:
    - other
---
### 局域网中的文件传输
>在同一个局域网中,如果方便快捷的传输文件?
如果你要想着搭建一个ftp服务器,就太麻烦了. 现在一个命令就可以解决

### 开启http服务器
1. 在Python3的环境下,直接运行
```py
python -m http.server  # 默认开启8000端口
```
2. 在python2的环境下,直接运行
```py
python -m SimpleHTTPServer  # 默认开启8000端口
```

**现在，您可以访问http://localhost:8080来查看您的服务器**
这里会展示出来你开启该服务路径的文件夹内容,所有你需要传输哪个文件,就在相关的文件夹中开启就可以了

### 参数介绍
可选配置：
- -p 要使用的端口（默认为8000）
- -a 要使用的地址（默认为0.0.0.0）
- -d 显示目录列表（默认为“True”）
- -i 显示autoIndex（默认为“True”）
- -g或--gzip启用时（默认为“False”），它将用于./public/some-file.js.gz代替./public/some-file.jsgzip压缩版本的文件，并且该请求接受gzip编码。
- -e或--ext默认文件扩展名（如果没有提供）（默认为'html'）
- -s或--silent从输出中抑制日志消息
- --cors通过Access-Control-Allow-Origin标题启用CORS
- -o 启动服务器后打开浏览器窗口
- -c设置缓存控制max-age头的缓存时间（以秒为单位），例如-c10 10秒（默认为'3600'）。要禁用缓存，请使用-c-1。
- -U或--utc在日志消息中使用UTC时间格式。
- -P或--proxy代理无法在本地解决给定网址的所有请求。例如：-P http://someurl.com
- -S或--ssl启用https。
- -C或--certssl证书文件的路径（默认值：cert.pem）。
- -K或--keyssl密钥文件的路径（默认值：key.pem）。
- -r或者--robots提供一个/robots.txt（其内容默认为'User-agent：* \ nDisallow：/'）
- -h或--help打印此列表并退出。


### 流程展示
![](https://raw.githubusercontent.com/shen-wanjiang/save_picture/master/markdown_pic/http-server.png)

**我的电脑在局域网中的ip为:192.168.1.251, 我开启了默认的8000端口**

我用同一个局域网的电脑或者手机客户端访问: http://192.168.1.251:8000
就可以直接下载文件
![](https://raw.githubusercontent.com/shen-wanjiang/save_picture/master/markdown_pic/http-server-phone.png)


### 手机端开启一个服务器(ftp)
在手机的文件管理中,有一个远程管理,可以直接一键开启一个ftp服务器,可以在其它设备上直接访问下载手机中的文件. 
![](https://raw.githubusercontent.com/shen-wanjiang/save_picture/master/markdown_pic/http-server-ftp.png)

**注意此时开启的是ftp服务器,所有,你访问的地址的协议要写对: ftp://xxx**

这样的话,无论从手机还是电脑它们之间的互传文件都非常的方便了.