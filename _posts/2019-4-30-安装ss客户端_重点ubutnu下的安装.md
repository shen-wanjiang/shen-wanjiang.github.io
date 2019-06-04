---
layout:     post
title:      安装ss客户端
subtitle:   重点ubutnu下的安装
date:       2019-04-30
author:     WJ
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - shadowsocks
---
> 搭建好ss服务器之后, 我们还需要在客户端配置相关的数据,这样就可以直接连接我们的ss服务器,畅游网络了.

# 1. ss客户端

1. [Windows下的Shadowsocks客户端](https://github.com/shadowsocks/shadowsocks-windows/releases)

2. [Mac下的Shadowsocks客户端](https://github.com/shadowsocks/ShadowsocksX-NG/releases)

3. Linux中的Shadowsocks客户端
    1. [GUI Client: Shadowsocks-Qt5](https://github.com/shadowsocks/shadowsocks-qt5/wiki/Installation)

4. [Android中的Shadowsocks客户端](https://github.com/shadowsocks/shadowsocks-android/releases)

# ubutnu下ss安装与配置(命令行)
1. 安装ss
    ```shell
    sudo apt-get update 
    sudo apt-get install python-gevent python-pip
    pip install shadowsocks
    ```

2. 配置文件的两种方法.
    1. 方法一: **直接输入命令运行**
        终端输入 `sslocal -help` 可以看到帮助文件，如图：
        ![如图](https://raw.githubusercontent.com/shen-wanjiang/save_picture/master/markdown_pic/ss%E5%AE%A2%E6%88%B7%E7%AB%AF-help.png)

        命令如下:
        `sslocal -s 1.1.1.1 -p 2019 -k "your passwd" -b 127.0.0.1 -l 1080`
        参数如下:
        - -s后面跟你的服务器ip 
        - -p后面跟你服务器远程端口号(我的是2019) 
        - -k后面跟你的密码（写在双引号之间）
        - 其他的用默认选项就好（想改的参见帮助文档）

    2. 方法二: **文件读取运行(我采用的是这种)** 
        在你的~目录下新建一个.json文件（或者别的地方，都可以）
        touch shadowsocks.json /etc/shadowsocks.json　　# 我的文件放在这里，请按照自己实际情况新建文件
        ok，不管怎么样，现在我们有了一个.json的文件，然后打开编辑，内容如下：
        ```json
        {
        "server":"1.1.1.1",
        "server_port":2019,
        "local_address": "127.0.0.1",
        "local_port":1080,
        "password":"your passwd",
        "timeout":300,
        "method":"aes-256-cfb"
        }
        ```
        其中，
        - server填你的服务器ip，
        - sever_port填远程端口号，
        - local_address本地ip，
        - local_part本地端口(也就是你本地一会开启服务的端口)，
        - password填密码，
        - timeout是延迟时间，
        - method是加密方式，
        - 按照实际情况填写并保存。

3. 运行ss
    保存完运行如下命令（路径以实际为准）：
    ```shell
    sslocal -c /home/shen/ss.json
    ```
    出现下图所示,即为成功.
    ![](https://raw.githubusercontent.com/shen-wanjiang/save_picture/master/markdown_pic/ss%E5%90%AF%E5%8A%A8%E6%88%90%E5%8A%9F.png)

    如果你不慎关掉,重新运行,发现端口被占用了. 用下面方法处理:
    lsof -i :1080
    得到数据后,你发现占用端口的pid后
    kill -9 pid    # 直接杀掉,然后启动就可以了

4. 设置开机启动
    ```shell
    # 打开图形化开机启动项管理界面
    gnome-session-properties
    # 添加(Add) -> 名称(name)和描述(comment)随便填，命令(Command)填写如下： 
    sslocal -c /home/shen/ss.json
    # 搞定
    ```

# 设置代理
以上就是SS的搭建了，这个时候我们发现上网时并不可以翻墙，原因是需要将sock5代理映射为http代理。
代理的软件很多，我选择了推荐度比较高的privoxy，下面是privoxy的配置。
#### 终端设置(要想终端连接外网)
1. 安装 privoxy
    ```shell
    sudo apt install privoxy
    ```
2. 配置privoxy
    打开 `/etc/privoxy/config`
    ```shell
    sudo vi /etc/privoxy/config
    ```
    找到其中的4.1节，看一下有没有一句listen-address localhost:8118的代码，如果被注释了，取消注释。因为版本不一样这句的状态可能会不一样。 
    然后再将 localhost 改成 127.0.0.1（这一步很重要，反正我因为这一步的设置问题搞了很久都不知道为什么连不上外网），如图所示：
    ![](https://raw.githubusercontent.com/shen-wanjiang/save_picture/master/markdown_pic/privoxy_1.png)

    接着找到5.2节，在本节末尾加入代码 forward-socks5t / 127.0.0.1:1080 .，注意最后有一个点号，如图：
    ![](https://raw.githubusercontent.com/shen-wanjiang/save_picture/master/markdown_pic/privoxy_2.png)

3. 重启 privoxy 服务
    ```shell
    sudo /etc/init.d/privoxy restart
    ```

4. 设置开机自启动 privoxy 服务
    将启动服务的命令添加到 /etc/rc.local 文件中的 exit 0 之前：
    ```shell
    sudo vi /etc/rc.local
    ……
    /etc/init.d/privoxy start
    exit 0
    ```

5. 代理配置
    ```shell
    sudo vi /etc/profile   # 这个文件是每个用户登录时都会运行的环境变量设置
    # 将下面两行加入文件中
    export http_proxy=http://127.0.0.1:8118
    export https_proxy=http://127.0.0.1:8118
    # 更新
    $ source /etc/profile
    ```

6. 测试
    `wget www.google.com`

7. 说一下流程
    首先我们启动了代理profile. 在配置文件中,它启动的默认端口是8118,   
    然后和我们的ss关联,配置了 `forward-socks5t / 127.0.0.1:1080.`   
    然后,我们设置http_proxy. 如: export http_proxy=http://127.0.0.1:8118   
    这样,我们在终端发送请求,先被导入到8118这个privoxy代理,然后该代理再调用ss客户,然后该ss客户端连接到国外的ss服务器,返回我们需要的数据.   
    这样,我们的流程就走完了  
8. 取消终端的代理
    因为不能像浏览器一样设置智能的切换.所以这里的终端都是代理服务
    你应该使用 `unset http_proxy` 和 `unset https_proxy`来取消代理
    如果你直接将 privoxy 服务停止, 这样是不行的.因为你的终端第一层代理已经指向了 privoxy 的1080端口,你把这个服务停止,就会访问出错.
    正确的做法是,直接将第一层的 http 代理取消.即使这个privoxy开着服务,你也不会调用,而是直接走正常的流程

9. 说明可能出现的问题
    1. **本地git代码拉取不下来**
    我这里配置好之后,发现,我拉取内网git(192.168.1.136)的时候,死活拉取不下来了.
    ping这个网址还是可以ping通的. 后来就思考了一下7的流程.直接把全局的代理关闭就可以了.
    **Linux - 设置/取消代理**
        - 设置
        export http_proxy=127.0.0.1:8118
        或者
        export https_proxy=127.0.0.1:8118
        - 取消该设置
        unset http_proxy
        或者
        unset https_proxy

    2. 全局git的配置
    我们设置的代理,如: export http_proxy=127.0.0.1:8118
    这样配置,有人说对于git的请求是无效的,但是我设置代理后,本地git就拉取不来,说明还是有效
    所有,要想给git的请求用上代理,我们需要设置git
        **git设置代理**
        ```shell
        git config --global https.proxy http://127.0.0.1:1080
        git config --global https.proxy https://127.0.0.1:1080
        ```
        **git取消代理**
        ```shell
        git config --global --unset http.proxy
        git config --global --unset https.proxy
        ```

    基于上述问题,开始我还以为是git问题,就把git配置上代理,等找到问题后,还是不行,才想到我已经给git配置了代理,所有,取消掉, 终于可以拉取本地代码了.
    当然,你如果想终端翻墙,就再次设置 export http_proxy=127.0.0.1:8118 就可以了


#### 浏览器设置
**准备**
- chrome
- 已经配置好了的ss

**操作**
1. Chrome 商店搜索SwtichyOmega，选择Proxy SwtichyOmega下载并安装扩展。(暂时登录不了的,请自行搜索插件安装)
2. 进入SwithyOmega进行设置:
对proxy的代理协议，代理服务器，代理端口进行设置。
一般对于Shawdowsocks用户，设置为：代理协议SOCKS5，代理服务器localhost，代理端口1080；
对auto switch进行设置，点击添加规则列表，规则列表设置改为AutoProxy，情景模式选择proxy，规则列表网址设置为
`https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt`  # 这里的文件,就是PAC(Proxy auto-config: 代理自动配置),记录哪些网址是国内的不需要使用代理,那些是需要代理的
    1. 点击左侧应用选项进行保存。
    2. 在浏览器插件位置把Proxy SwitchyOmega设置成anto switch。
    3. 访问Google试验一下~