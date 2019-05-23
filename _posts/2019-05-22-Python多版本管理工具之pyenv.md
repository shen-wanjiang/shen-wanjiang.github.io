---
layout:     post
title:      Python多版本管理工具之pyenv
subtitle:   pyenv配置及使用
date:       2019-05-22
author:     WJ
header-img: img/post-bg-python-unicode.jpg
catalog: true
tags:
    - other
---
### 使用pyenv管理Python版本 
[pyenv](https://github.com/pyenv/pyenv) 是 Python版本管理工具. pyenv 可以改变全局的 Python 版本，安装多个版本的 Python， 设置目录级别的 Python 版本，还能创建和管理虚拟环境( virtual python environments). 所有的设置都是用户级别的操作，不需要 sudo 命令.

pyenv 主要用来管理 Python 的版本，比如一个项目需要 Python 2.x ，一个项目需要 Python 3.x.
virtualenv 主要用来管理 Python 包的依赖，不同项目需要依赖的包版本不同，则需要使用虚拟环境.

原理:
pyenv 通过系统修改环境变量来实现 Python 不同版本的切换。
virtualenv 通过将 Python 包安装到一个目录来作为 Python 包虚拟环境，通过切换目录来实现不同包环境间的切换。

pyenv 的美好之处在于，它并没有使用将不同的PATH植入不同的shell这种高耦合的工作方式,而是简单的在PATH的最前面插入了一个垫片路径(shims):`~/.pyenv/shims:/usr/local/bin:/usr/bin:/bin`.所有对Python可执行文件的查找都会首先被这个shims路径截获,从而使后方的系统路径失效

### pyenv 安装
1. 针对不同系统,安装不同的工具
[Common build problems](https://github.com/pyenv/pyenv/wiki/Common-build-problems)

    如果你是mac系统,请直接使用[Homebrew](https://brew.sh/)来安装
    [具体操作](https://github.com/pyenv/pyenv#homebrew-on-macos):
    ```sehll
    $ brew update
    $ brew install pyenv
    ```
    安装之后请跳过第二部,直接添加环境变量

2. pyenv安装
pyenv的安装有两种方式:  
    1. **自动安装(推荐)**
    pyenv 提供了自动安装的工具，执行命令安装即可：
    `curl https://pyenv.run | bash`
    保证系统有 git ，否则需要新安装 git. 如果curl命令有问题,见下方的问题解答
    2. 手动安装(手动安装可以更加详细的了解安装过程，请参考[Installation](https://github.com/pyenv/pyenv#installation))
        1. git clone https://github.com/pyenv/pyenv ~/.pyenv
        2. 将PYENV_ROOT和pyenv init加入bash的~/.bashrc
            ```shell
            1 echo 'export PATH=~/.pyenv/bin:$PATH' >> ~/.bashrc
            2 echo 'export PYENV_ROOT=~/.pyenv' >> ~/.bashrc
            3 echo 'eval "$(pyenv init -)"' >> ~/.bashrc
            4 source ~/.bashrc
            ```
        3. 安装pyenv-virtualenv(**注意**, 手动安装不会安装虚拟环境相关控件:pyenv-virtualenv)
            ```shell
            1 git clone https://github.com/pyenv/pyenv-virtualenv ~/.pyenv/plugins/pyenv-virtualenv
            2 echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.bash_profile
            3 source ~/.bashrc
            ```
        

3. 添加环境变量
    安装成功后记得在 .bashrc 或者 .bash_profile 中添加三行来开启自动补全。
    ```shell
    export PATH="$HOME/.pyenv/bin:$PATH"
    eval "$(pyenv init -)"
    eval "$(pyenv virtualenv-init -)"
    ```
    然后使用命令刷新终端:
    ```shell
    source ~/.bashrc
    ```

### pyenv 常用命令
使用 `pyenv commands` 显示所有可用命令
**1. 查看本机安装 Python 版本**
```shell
pyenv versions
```
星号表示当前正在使用的 Python 版本。
**2. 查看可安装 Python 版本**
```shell
pyenv install -l
```
**3. python版本的安装和卸载**
```shell
$ pyenv install 2.7.3   # 安装 python
$ pyenv uninstall 2.7.3 # 卸载 python
```
**4. python 切换**
```shell
$ pyenv global 2.7.3  # 设置全局的 Python 版本，通过将版本号写入 ~/.pyenv/version 文件的方式。
$ pyenv local 2.7.3 # 设置 Python 本地版本，通过将版本号写入当前目录下的 .python-version 文件的方式。通过这种方式设置的 Python 版本优先级较 global 高。
```

**5. python 优先级设置和取消**
Python优先级
`shell > local > global`
pyenv 会从当前目录开始向上逐级查找 .python-version 文件，直到根目录为止。若找不到，就用 global 版本。
- 设置优先级
    ```shell
    $ pyenv shell 2.7.3 # 设置面向 shell 的 Python 版本，通过设置当前 shell 的 PYENV_VERSION 环境变量的方式。这个版本的优先级比 local 和 global 都要高。–unset 参数可以用于取消当前 shell 设定的版本。

    $ pyenv rehash  # 创建垫片路径（为所有已安装的可执行文件创建 shims，如：~/.pyenv/versions/*/bin/*，因此，每当你增删了 Python 版本或带有可执行文件的包（如 pip）以后，都应该执行一次本命令）
    ```
- 取消优先级
    如果你设置当前环境的python版本后,想取消,可以使用 --unset参数
    ```shell
    $ pyenv shell --unset  # 如果设置了 pyenv shell 3.6.0 后,想要取消该设置,用这个命令
    $ pyenv local --unset  # 如果设置了 pyenv local 3.6.0 后,想要取消该设置,用这个命令
    $ pyenv global --unset  # 如果设置了 pyenv global 3.6.0 后,想要取消该设置,用这个命令
    ```
**6. 常用命令总结**
```shell
pyenv install --list # 列出可安装版本
pyenv install <version> # 安装对应版本
pyenv install -v <version> # 安装对应版本，若发生错误，可以显示详细的错误信息
pyenv versions # 显示当前使用的python版本
pyenv which python # 显示当前python安装路径
pyenv global <version> # 设置默认Python版本
pyenv local <version> # 当前路径创建一个.python-version, 以后进入这个目录自动切换为该版本
pyenv shell <version> # 当前shell的session中启用某版本，优先级高于global 及 local
```

### 设置虚拟环境: pyenv-virtualenv
pyenv 插件：pyenv-virtualenv

使用自动安装 pyenv 后，它会自动安装部分插件，通过 pyenv-virtualenv 插件可以很好的和 virtualenv 结合：

**1. 创建虚拟环境**
```shell
pyenv virtualenv 2.7.10 env-2.7.10  # 2.7.10为用pyenv安装的python版本, 后面为基于该版本创建的虚拟环境的名字(自定义名字)
```
若不指定 python 版本，会默认使用当前环境 python 版本。如果指定 Python 版本，则一定要是已经安装过的版本(如已经安装的Python版本2.7.10)，否则会出错。
**环境的真实目录位于 ~/.pyenv/versions 下**

**2. 列出当前所有的虚拟环境**
```shell
pyenv virtualenvs
```

**3. 激活当前虚拟环境**
```shell
pyenv activate env-name  # 激活虚拟环境
```

**4. 退出当前虚拟环境**
```shell
pyenv deactivate #退出虚拟环境，回到系统环境
```

**5. 删除虚拟环境**
```shell
pyenv uninstall my-virtual-env  # 后面为你虚拟环境的名字
```
或者删除其真实目录
```shell
rm -rf ~/.pyenv/versions/env-name  # 删除其真实目录
```

使用 pyenv 来管理 python，使用 pyenv-virtualenv 插件来管理多版本 python 包。
此时，还需注意，当我们将项目运行的 env 环境部署到生产环境时，由于我们的 python 包是依赖 python 的，需要注意生产环境的 python 版本问题。

### 安装的python版本以及虚拟环境的位置
以上面创建虚拟环境为例子: `pyenv virtualenv 2.7.10 env-2.7.10`
如果我们想在其它项目中引用这个环境怎么办?它包含了我们的python版本以及我们下载的相关第三方包

**它的路径在: `~/.pyenv/versions/env_2.7.10/bin/python`**
如果你安装的.pyenv位置或者虚拟环境的名字不一样,同样的格式,查找即可
当然,如果仅仅要我们安装的python版本而不要虚拟环境,也是同样的位置,在versions下会有2.7.10,就是你的python解释器

### 安装过程中出现的错误
1. 在服务器上使用 curl命令出现错误: `curl: (35) SSL connect error`
无法在服务器使用curl命令访问https域名,原因是nss版本有点旧了，yum -y update nss更新一下，重新curl即可！

2. 错误:`BUILD FEILED (Ubuntu 16.04 using python-build 1.2.2)`
问题是缺少依赖包(第一步没有安装所需要的包)，不同系统见以下链接:
`https://github.com/pyenv/pyenv/wiki/Common-build-problems`